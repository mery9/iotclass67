# Analyze and make aggregations.
หมายถึงกระบวนการวิเคราะห์และสรุปข้อมูล โดย Analyze คือการตรวจสอบและแยกแยะข้อมูลเพื่อหาความสัมพันธ์หรือรูปแบบบางอย่าง ส่วน Aggregations คือการรวบรวมข้อมูล เช่น การนับ ค่าเฉลี่ย ยอดรวม หรือการจัดกลุ่มข้อมูล เพื่อนำเสนอข้อมูลในภาพรวมที่เข้าใจง่ายและมีความหมายมากขึ้น

## Aggregate Metrics By Sensor Processor

หลักการทำงาน: ส่วนนี้จะรวบรวมข้อมูลจากเซนเซอร์แต่ละตัว เช่น เซนเซอร์ตรวจจับความชื้น, อุณหภูมิ หรือเซนเซอร์อื่นๆ ที่ติดตั้งในระบบ โดยข้อมูลจากเซนเซอร์แต่ละตัวจะถูกนำมาประมวลผลรวมกัน (aggregate) เพื่อคำนวณค่าต่างๆ ที่เกี่ยวข้องกับเซนเซอร์ เช่น ค่าเฉลี่ย, ค่าสูงสุด, ค่าต่ำสุด หรือค่าเบี่ยงเบนมาตรฐานในแต่ละช่วงเวลา

ทำแล้วได้อะไร: การรวบรวมและคำนวณนี้ช่วยให้สามารถติดตามประสิทธิภาพของเซนเซอร์ในแต่ละตัวได้อย่างใกล้ชิด เช่น ถ้าค่าอุณหภูมิสูงเกินเกณฑ์จะสามารถแจ้งเตือนได้ทันที ทำให้การตัดสินใจดำเนินการต่างๆ เช่น ซ่อมบำรุง หรือปรับการทำงานของระบบทำได้รวดเร็วและมีประสิทธิภาพมากขึ้น

```
@Component
public class AggregateMetricsBySensorProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsBySensorProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-sensor-tmp";

    /**
     * Agg Metrics Sensor Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-sensor}")
    private String aggMetricsSensorOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetricsBySensor(stream)
                .to(aggMetricsSensorOutput, Produced.with(String(), new SensorAggregateMetricsSensorSerde()));
    }

    /**
     * Build Aggregate Metrics By Sensor Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregateSensorMetricsDTO> buildAggregateMetricsBySensor(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregateSensorMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregateSensorMetricsDTO va) -> aggregateData(v, va),
                         buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsSensorSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregateSensorMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregateSensorMetricsDTO va) {
        // Sensor Data
        va.setId(v.getId());
        // Sensor Data
        va.setId(v.getId());
        va.setName(v.getName());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
```

## Aggregate Metrics By Place Processor

หลักการทำงาน: ส่วนนี้จะทำหน้าที่รวบรวมข้อมูลที่ได้จากเซนเซอร์ทั้งหมดที่ติดตั้งในสถานที่หรือพื้นที่ที่กำหนด เช่น สำนักงาน, โรงงาน หรือพื้นที่สาธารณะ จากนั้นจะนำข้อมูลทั้งหมดมาสรุปผลเป็นสถิติของพื้นที่นั้นๆ

ทำแล้วได้อะไร: การรวบรวมข้อมูลในระดับพื้นที่ช่วยให้สามารถตรวจสอบประสิทธิภาพหรือสถานะของพื้นที่ที่กำหนดได้อย่างชัดเจน เช่น ถ้าอุณหภูมิในพื้นที่หนึ่งสูงขึ้นมาก จะสามารถระบุปัญหาและแก้ไขได้รวดเร็ว รวมถึงสามารถใช้ข้อมูลนี้ในการปรับปรุงประสิทธิภาพการใช้งานพลังงานหรือการรักษาความปลอดภัยในพื้นที่นั้นๆ
```
@Component
public class AggregateMetricsByPlaceProcessor {

    private static final Logger logger = LoggerFactory.getLogger(AggregateMetricsByPlaceProcessor.class);

    private final static int WINDOW_SIZE_IN_MINUTES = 5;
    private final static String WINDOW_STORE_NAME = "aggregate-metrics-by-place-tmp";

    /**
     * Agg Metrics Place Topic Output
     */
    @Value("${kafka.topic.aggregate-metrics-place}")
    private String aggMetricsPlaceOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetrics(stream)
                .to(aggMetricsPlaceOutput, Produced.with(String(), new SensorAggregateMetricsPlaceSerde()));
    }

    /**
     * Build Aggregate Metrics Stream
     *
     * @param stream
     * @return
     */
    private KStream<String, SensorAggregatePlaceMetricsDTO> buildAggregateMetrics(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getPlaceId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)).grace(Duration.ofMillis(0)))
                .aggregate(SensorAggregatePlaceMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregatePlaceMetricsDTO va) -> aggregateData(v, va),
                        buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    /**
     * Build Window Persistent Store
     *
     * @return
     */
    private Materialized<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>>as(WINDOW_STORE_NAME)
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsPlaceSerde());
    }

    /**
     * Aggregate Data
     *
     * @param v
     * @param va
     * @return
     */
    private SensorAggregatePlaceMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregatePlaceMetricsDTO va) {
        va.setPlaceId(v.getId());
        // Start Agg
        if (va.getStartAgg() == null) {
            final Date startAggAt = new Date();
            va.setStartAgg(startAggAt);
            va.setStartAggTm(startAggAt.getTime());
        }
        va.setCountMeasures(va.getCountMeasures() + 1);
        // Temperature
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures()); // Humidity
        // Humidity
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures()); // Luminosity
        // Luminosity
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures()); // Pressure
        // Pressure
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());

        // End Agg
        final Date endAggAt = new Date();
        va.setEndAgg(endAggAt);
        va.setEndAggTm(endAggAt.getTime());
        return va;
    }

}
```

## Aggregate Metrics time series

หลักการทำงาน: ส่วนนี้จะทำการรวบรวมและจัดเก็บข้อมูลที่ได้รับจากเซนเซอร์ในรูปแบบของ "Time Series" หรือข้อมูลที่มีการบันทึกตามลำดับเวลาที่ต่อเนื่อง โดยจะสร้างเป็นกราฟเวลาที่แสดงให้เห็นแนวโน้มของข้อมูลต่างๆ ที่เกิดขึ้นในช่วงเวลาที่กำหนด

ทำแล้วได้อะไร: การมีข้อมูล Time Series ทำให้สามารถวิเคราะห์แนวโน้มของการเปลี่ยนแปลงของข้อมูลในระยะยาวได้อย่างชัดเจน เช่น การติดตามการเปลี่ยนแปลงของอุณหภูมิในช่วงสัปดาห์ที่ผ่านมา จะช่วยให้คาดการณ์ปัญหาล่วงหน้า และสามารถวางแผนการดำเนินงานได้อย่างมีประสิทธิภาพมากขึ้น เช่น การปรับปรุงระบบทำความเย็น หรือการบำรุงรักษาระบบที่เกี่ยวข้องกับเซนเซอร์
```
@Component
public class MetricsTimeSeriesProcessor {

    private static final Logger logger = LoggerFactory.getLogger(MetricsTimeSeriesProcessor.class);

    private final static String SENSOR_TIME_SERIE_NAME = "sample_sensor_metric";
    private final static String SENSOR_TIME_SERIE_TYPE = "sensor";

    /**
     * Metrics Time Series
     */
    @Value("${kafka.topic.metrics-time-series}")
    private String metricTimeSeriesOutput;

    /**
     *
     * @param stream
     */
    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        stream
                .map((key, val) -> new KeyValue<>(val.getId(), buildSensorTimeSerieMetric(val)))
                .to(metricTimeSeriesOutput, Produced.with(String(), new SensorTimeSerieMetricSerde()));
    }

    /**
     * Build Sensor Time Serie Meter
     *
     * @param sensorData
     * @return
     */
    private SensorTimeSerieMetricDTO buildSensorTimeSerieMetric(final SensorDataDTO sensorData) {
        return SensorTimeSerieMetricDTO.builder()
                .name(SENSOR_TIME_SERIE_NAME)
                .timestamp(new Date().getTime())
                .type(SENSOR_TIME_SERIE_TYPE)
                .dimensions(SensorTimeSerieMetricDimensionsDTO.builder()
                        .placeId(sensorData.getPlaceId())
                        .sensorId(sensorData.getId())
                        .sensorName(sensorData.getName())
                        .build())
                .values(SensorTimeSerieMetricValuesDTO.builder()
                        .humidity((double) sensorData.getPayload().getHumidity())
                        .luminosity((double) sensorData.getPayload().getLuminosity())
                        .pressure((double) sensorData.getPayload().getPressure())
                        .temperature((double) sensorData.getPayload().getTemperature())
                        .build())
                .build();
    }

}
```
