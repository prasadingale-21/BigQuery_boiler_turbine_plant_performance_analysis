# Boiler Turbine Plant Performance Analysis

End-to-end industrial plant performance analytics project using **Google BigQuery, SQL and Data Studio**.

**Workflow:** Kaggle dataset → Google Cloud → BigQuery → SQL analysis → Data Studio dashboard → business insights

## Business objectives
- Profile and validate industrial sensor data.
- Analyze fuel, steam, boiler efficiency and generator performance.
- Evaluate turbine and condenser operating conditions.
- Examine environmental relationships.
- Screen for potential statistical anomalies.
- Present findings through an executive/operational dashboard.

> Correlation indicates statistical association in the observed data; it does not establish causation.

## Technology stack
| Tool | Use |
|---|---|
| Google BigQuery | Cloud warehouse and SQL analytics |
| Data Studio | Interactive dashboard |
| SQL | Profiling, quality, statistics, relationships and insights |
| GitHub | Portfolio/version control |
| Kaggle | Source dataset |

## Dataset
**Boiler Turbine Plant Sensor Data** — Kaggle:
https://www.kaggle.com/datasets/drsayed/boiler-turbine-plant-sensor-data

120,000 rows and 25 columns.

## BigQuery table
`boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`


## SQL workflow

-- Data Understanding 

  ### How many rows are there?   
  
    SELECT COUNT(*) FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`; 

  ### What columns do we have?  
  
    SELECT 
      column_name, 
      data_type, 
      is_nullable
    FROM 
      `boiler-turbine-plant-analytics.raw_data.INFORMATION_SCHEMA.COLUMNS`
    WHERE 
      table_name = 'boiler_turbine_sensor_data'
    ORDER BY 
      ordinal_position;

### Are there duplicate rows?
  
      SELECT
        Feedwater_Temp_C, Feedwater_Pressure_bar, Feedwater_Flow_m3h,Fuel_Flow_kg_s,Combustion_Air_Flow_km3h, COUNT(*)
      FROM 
        `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
      GROUP BY
        Feedwater_Temp_C, Feedwater_Pressure_bar, Feedwater_Flow_m3h, Fuel_Flow_kg_s,Combustion_Air_Flow_km3h
      HAVING
        COUNT(*)>1;


  ### What does the data look like?
  
      SELECT *
      FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
      LIMIT 10;

  ### Missing-value analysis
  
        SELECT
          COUNT(*) AS total_rows,
        
          COUNTIF(Feedwater_Temp_C IS NULL) AS missing_feedwater_temp,
          COUNTIF(Feedwater_Pressure_bar IS NULL) AS missing_feedwater_pressure,
          COUNTIF(Feedwater_Flow_m3h IS NULL) AS missing_feedwater_flow,
          COUNTIF(Fuel_Flow_kg_s IS NULL) AS missing_fuel_flow,
          COUNTIF(Combustion_Air_Flow_km3h IS NULL) AS missing_combustion_air,
          COUNTIF(Flue_Gas_Temp_C IS NULL) AS missing_flue_gas_temp,
          COUNTIF(Flue_Gas_O2_pct IS NULL) AS missing_flue_gas_o2,
          COUNTIF(Steam_Drum_Pressure_bar IS NULL) AS missing_drum_pressure,
          COUNTIF(Steam_Drum_Level_pct IS NULL) AS missing_drum_level,
          COUNTIF(Superheater_Outlet_Temp_C IS NULL) AS missing_superheater_temp,
          COUNTIF(Superheater_Spray_Flow_m3h IS NULL) AS missing_spray_flow,
          COUNTIF(Turbine_Inlet_Pressure_bar IS NULL) AS missing_turbine_pressure,
          COUNTIF(Turbine_Inlet_Temp_C IS NULL) AS missing_turbine_temp,
          COUNTIF(Turbine_Speed_rpm IS NULL) AS missing_turbine_speed,
          COUNTIF(Generator_Power_MW IS NULL) AS missing_generator_power,
          COUNTIF(Condenser_Vacuum_kPa IS NULL) AS missing_condenser_vacuum,
          COUNTIF(Condenser_Cooling_Flow_m3h IS NULL) AS missing_cooling_flow,
          COUNTIF(Condenser_Hotwell_Temp_C IS NULL) AS missing_hotwell_temp,
          COUNTIF(Makeup_Water_Flow_m3h IS NULL) AS missing_makeup_water,
          COUNTIF(Ambient_Temperature_C IS NULL) AS missing_ambient_temp,
          COUNTIF(Ambient_Humidity_pct IS NULL) AS missing_humidity,
          COUNTIF(Barometric_Pressure_kPa IS NULL) AS missing_pressure,
          COUNTIF(Boiler_Efficiency_pct IS NULL) AS missing_efficiency,
          COUNTIF(Generator_Power_Factor IS NULL) AS missing_power_factor,
          COUNTIF(Steam_Flow_tonne_per_h IS NULL) AS missing_steam_flow
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


  ### Descriptive statistics
  
        SELECT
          COUNT(*) AS observations,
        
          MIN(Steam_Flow_tonne_per_h) AS min_steam_flow,
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(25)] AS p25_steam_flow,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(50)] AS median_steam_flow,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(75)] AS p75_steam_flow,
          MAX(Steam_Flow_tonne_per_h) AS max_steam_flow
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;

  ### Univariate Analysis
  
        SELECT
          'Steam Flow' AS parameter,
          COUNT(Steam_Flow_tonne_per_h) AS observations,
          MIN(Steam_Flow_tonne_per_h) AS min_value,
          AVG(Steam_Flow_tonne_per_h) AS mean_value,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(50)] AS median_value,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(25)] AS p25,
          APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(75)] AS p75,
          MAX(Steam_Flow_tonne_per_h) AS max_value
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        UNION ALL
        
        SELECT
          'Fuel Flow',
          COUNT(Fuel_Flow_kg_s),
          MIN(Fuel_Flow_kg_s),
          AVG(Fuel_Flow_kg_s),
          APPROX_QUANTILES(Fuel_Flow_kg_s, 100)[OFFSET(50)],
          APPROX_QUANTILES(Fuel_Flow_kg_s, 100)[OFFSET(25)],
          APPROX_QUANTILES(Fuel_Flow_kg_s, 100)[OFFSET(75)],
          MAX(Fuel_Flow_kg_s)
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        UNION ALL
        
        SELECT
          'Boiler Efficiency',
          COUNT(Boiler_Efficiency_pct),
          MIN(Boiler_Efficiency_pct),
          AVG(Boiler_Efficiency_pct),
          APPROX_QUANTILES(Boiler_Efficiency_pct, 100)[OFFSET(50)],
          APPROX_QUANTILES(Boiler_Efficiency_pct, 100)[OFFSET(25)],
          APPROX_QUANTILES(Boiler_Efficiency_pct, 100)[OFFSET(75)],
          MAX(Boiler_Efficiency_pct)
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        UNION ALL
        
        SELECT
          'Generator Power',
          COUNT(Generator_Power_MW),
          MIN(Generator_Power_MW),
          AVG(Generator_Power_MW),
          APPROX_QUANTILES(Generator_Power_MW, 100)[OFFSET(50)],
          APPROX_QUANTILES(Generator_Power_MW, 100)[OFFSET(25)],
          APPROX_QUANTILES(Generator_Power_MW, 100)[OFFSET(75)],
          MAX(Generator_Power_MW)
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        UNION ALL
        
        SELECT
          'Turbine Speed',
          COUNT(Turbine_Speed_rpm),
          MIN(Turbine_Speed_rpm),
          AVG(Turbine_Speed_rpm),
          APPROX_QUANTILES(Turbine_Speed_rpm, 100)[OFFSET(50)],
          APPROX_QUANTILES(Turbine_Speed_rpm, 100)[OFFSET(25)],
          APPROX_QUANTILES(Turbine_Speed_rpm, 100)[OFFSET(75)],
          MAX(Turbine_Speed_rpm)
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


### Bivariate analysis
  
### Does higher fuel flow correspond to higher steam production?
  
        SELECT
          Fuel_Flow_kg_s,
          Steam_Flow_tonne_per_h
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Fuel_Flow_kg_s IS NOT NULL
          AND Steam_Flow_tonne_per_h IS NOT NULL;

  ### Does steam flow correspond to generator power?
  
        SELECT
          Steam_Flow_tonne_per_h,
          Generator_Power_MW
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Steam_Flow_tonne_per_h IS NOT NULL
          AND Generator_Power_MW IS NOT NULL;

  ### Does turbine inlet pressure relate to steam flow?
  
        SELECT
          Turbine_Inlet_Pressure_bar,
          Steam_Flow_tonne_per_h
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;

  ### Correlation analysis
  
        SELECT
          CORR(Fuel_Flow_kg_s, Steam_Flow_tonne_per_h) AS fuel_vs_steam,
          CORR(Generator_Power_MW, Steam_Flow_tonne_per_h) AS power_vs_steam,
          CORR(Turbine_Inlet_Pressure_bar, Steam_Flow_tonne_per_h) AS pressure_vs_steam,
          CORR(Turbine_Inlet_Temp_C, Steam_Flow_tonne_per_h) AS temperature_vs_steam,
          CORR(Boiler_Efficiency_pct, Steam_Flow_tonne_per_h) AS efficiency_vs_steam,
          CORR(Condenser_Vacuum_kPa, Generator_Power_MW) AS vacuum_vs_power
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;

  ### Boiler Performance
  
        -- What is average boiler efficiency?
        -- How does fuel flow relate to efficiency?
        -- How does combustion air affect efficiency?
        -- How does flue-gas temperature relate to efficiency?
        -- How does flue-gas O₂ relate to efficiency?
        -- What conditions correspond to high efficiency?
        -- Can we identify inefficient operating conditions?
        
        SELECT
          AVG(Boiler_Efficiency_pct) AS avg_efficiency,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow,
          AVG(Combustion_Air_Flow_km3h) AS avg_air_flow,
          AVG(Flue_Gas_Temp_C) AS avg_flue_gas_temp,
          AVG(Flue_Gas_O2_pct) AS avg_flue_gas_o2,
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


  ### Turbine Performance
  
        -- Turbine inlet pressure
        -- Turbine inlet temperature
        -- Turbine speed
        -- Steam flow
        -- Generator power
        -- Condenser vacuum
        
        SELECT
          AVG(Turbine_Inlet_Pressure_bar) AS avg_inlet_pressure,
          AVG(Turbine_Inlet_Temp_C) AS avg_inlet_temperature,
          AVG(Turbine_Speed_rpm) AS avg_speed,
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Generator_Power_MW) AS avg_generator_power
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


  ### Generator Performance
  
        SELECT
          COUNT(*) AS observations,
          AVG(Generator_Power_MW) AS avg_generator_power,
          MIN(Generator_Power_MW) AS min_generator_power,
          MAX(Generator_Power_MW) AS max_generator_power,
          AVG(Generator_Power_Factor) AS avg_power_factor,
          MIN(Generator_Power_Factor) AS min_power_factor,
          MAX(Generator_Power_Factor) AS max_power_factor
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


  ### Steam Flow vs Generator Power
  
        SELECT
          Steam_Flow_tonne_per_h,
          Generator_Power_MW,
          Generator_Power_Factor
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Steam_Flow_tonne_per_h IS NOT NULL
          AND Generator_Power_MW IS NOT NULL;

  ### Generator power by steam-flow category
  
        SELECT
          CASE
            WHEN Steam_Flow_tonne_per_h < 80 THEN 'Low Steam Flow'
            WHEN Steam_Flow_tonne_per_h < 130 THEN 'Medium Steam Flow'
            ELSE 'High Steam Flow'
          END AS steam_flow_category,
        
          COUNT(*) AS observations,
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Generator_Power_MW) AS avg_generator_power,
          AVG(Generator_Power_Factor) AS avg_power_factor
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        WHERE Steam_Flow_tonne_per_h IS NOT NULL
        
        GROUP BY steam_flow_category
        ORDER BY avg_steam_flow;

  ### Condenser Performance
  ### Overall condenser statistics
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Condenser_Vacuum_kPa) AS avg_condenser_vacuum,
          MIN(Condenser_Vacuum_kPa) AS min_condenser_vacuum,
          MAX(Condenser_Vacuum_kPa) AS max_condenser_vacuum,
        
          AVG(Condenser_Cooling_Flow_m3h) AS avg_cooling_flow,
          MIN(Condenser_Cooling_Flow_m3h) AS min_cooling_flow,
          MAX(Condenser_Cooling_Flow_m3h) AS max_cooling_flow,
        
          AVG(Condenser_Hotwell_Temp_C) AS avg_hotwell_temperature,
          MIN(Condenser_Hotwell_Temp_C) AS min_hotwell_temperature,
          MAX(Condenser_Hotwell_Temp_C) AS max_hotwell_temperature
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


  ### Condenser vacuum vs generator power
  
        SELECT
          Condenser_Vacuum_kPa,
          Generator_Power_MW
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Condenser_Vacuum_kPa IS NOT NULL
          AND Generator_Power_MW IS NOT NULL;

  ### Cooling flow vs condenser vacuum
  
        SELECT
          Condenser_Cooling_Flow_m3h,
          Condenser_Vacuum_kPa,
          Condenser_Hotwell_Temp_C
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Condenser_Cooling_Flow_m3h IS NOT NULL
          AND Condenser_Vacuum_kPa IS NOT NULL;

  ### Correlation
  
        SELECT
          CORR(Condenser_Vacuum_kPa, Generator_Power_MW)
            AS vacuum_vs_generator_power,
        
          CORR(Condenser_Cooling_Flow_m3h, Condenser_Vacuum_kPa)
            AS cooling_flow_vs_vacuum,
        
          CORR(Condenser_Hotwell_Temp_C, Generator_Power_MW)
            AS hotwell_temp_vs_generator_power
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;


### Environmental Analysis
  
### Environmental statistics
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Ambient_Temperature_C) AS avg_ambient_temperature,
          MIN(Ambient_Temperature_C) AS min_ambient_temperature,
          MAX(Ambient_Temperature_C) AS max_ambient_temperature,
        
          AVG(Ambient_Humidity_pct) AS avg_humidity,
          MIN(Ambient_Humidity_pct) AS min_humidity,
          MAX(Ambient_Humidity_pct) AS max_humidity,
        
          AVG(Barometric_Pressure_kPa) AS avg_barometric_pressure,
          MIN(Barometric_Pressure_kPa) AS min_barometric_pressure,
          MAX(Barometric_Pressure_kPa) AS max_barometric_pressure
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;
  
  ### Environment vs Steam Flow
  
        SELECT
          Ambient_Temperature_C,
          Ambient_Humidity_pct,
          Barometric_Pressure_kPa,
          Steam_Flow_tonne_per_h
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Steam_Flow_tonne_per_h IS NOT NULL;


  ### Environment vs Generator Power
  
        SELECT
          Ambient_Temperature_C,
          Ambient_Humidity_pct,
          Barometric_Pressure_kPa,
          Generator_Power_MW
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Generator_Power_MW IS NOT NULL;

  ### Correlation analysis
  
        SELECT
          CORR(Ambient_Temperature_C, Steam_Flow_tonne_per_h)
            AS temperature_vs_steam,
        
          CORR(Ambient_Humidity_pct, Steam_Flow_tonne_per_h)
            AS humidity_vs_steam,
        
          CORR(Barometric_Pressure_kPa, Steam_Flow_tonne_per_h)
            AS pressure_vs_steam,
        
          CORR(Ambient_Temperature_C, Generator_Power_MW)
            AS temperature_vs_power,
        
          CORR(Ambient_Humidity_pct, Generator_Power_MW)
            AS humidity_vs_power,
        
          CORR(Barometric_Pressure_kPa, Generator_Power_MW)
            AS pressure_vs_power
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;

  ### Operating-condition segmentation
  
        SELECT
          CASE
            WHEN Steam_Flow_tonne_per_h < 80 THEN 'Low'
            WHEN Steam_Flow_tonne_per_h < 130 THEN 'Medium'
            ELSE 'High'
          END AS steam_flow_category,
        
          COUNT(*) AS observations,
          AVG(Boiler_Efficiency_pct) AS avg_efficiency,
          AVG(Generator_Power_MW) AS avg_generator_power,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        GROUP BY steam_flow_category
        ORDER BY
          CASE steam_flow_category
            WHEN 'Low' THEN 1
            WHEN 'Medium' THEN 2
            WHEN 'High' THEN 3
          END;


  ### Anomaly analysis
  
  ### Detect unusual Steam Flow
  
        WITH quartiles AS (
          SELECT
            APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(25)] AS q1,
            APPROX_QUANTILES(Steam_Flow_tonne_per_h, 100)[OFFSET(75)] AS q3
          FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        ),
        
        limits AS (
          SELECT
            q1,
            q3,
            q3 - q1 AS iqr,
            q1 - 1.5 * (q3 - q1) AS lower_limit,
            q3 + 1.5 * (q3 - q1) AS upper_limit
          FROM quartiles
        )
        
        SELECT
          t.Steam_Flow_tonne_per_h,
          t.Fuel_Flow_kg_s,
          t.Boiler_Efficiency_pct,
          t.Generator_Power_MW,
        
          CASE
            WHEN t.Steam_Flow_tonne_per_h < l.lower_limit
              THEN 'Potentially Low Anomaly'
        
            WHEN t.Steam_Flow_tonne_per_h > l.upper_limit
              THEN 'Potentially High Anomaly'
        
            ELSE 'Normal'
          END AS steam_flow_status
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data` t
        CROSS JOIN limits l
        WHERE t.Steam_Flow_tonne_per_h IS NOT NULL;


  ### Detect unusual Boiler Efficiency
  
        WITH quartiles AS (
          SELECT
            APPROX_QUANTILES(Boiler_Efficiency_pct, 100)[OFFSET(25)] AS q1,
            APPROX_QUANTILES(Boiler_Efficiency_pct, 100)[OFFSET(75)] AS q3
          FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        ),
        
        limits AS (
          SELECT
            q1,
            q3,
            q3 - q1 AS iqr,
            q1 - 1.5 * (q3 - q1) AS lower_limit,
            q3 + 1.5 * (q3 - q1) AS upper_limit
          FROM quartiles
        )
        
        SELECT
          t.Boiler_Efficiency_pct,
          t.Fuel_Flow_kg_s,
          t.Combustion_Air_Flow_km3h,
          t.Flue_Gas_Temp_C,
          t.Flue_Gas_O2_pct,
          t.Steam_Flow_tonne_per_h,
        
          CASE
            WHEN t.Boiler_Efficiency_pct < l.lower_limit
              THEN 'Potentially Low Efficiency Anomaly'
        
            WHEN t.Boiler_Efficiency_pct > l.upper_limit
              THEN 'Potentially High Efficiency Anomaly'
        
            ELSE 'Normal'
          END AS efficiency_status
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data` t
        CROSS JOIN limits l
        WHERE t.Boiler_Efficiency_pct IS NOT NULL;

  ### Find potentially inefficient operating conditions
  
        We can identify observations where:
        
        * Fuel consumption is relatively high  
        * But steam production is relatively low  
        * And boiler efficiency is relatively low  
        
        SELECT
          Fuel_Flow_kg_s,
          Steam_Flow_tonne_per_h,
          Boiler_Efficiency_pct,
          Generator_Power_MW,
          Flue_Gas_Temp_C,
          Flue_Gas_O2_pct,
        
          CASE
            WHEN Fuel_Flow_kg_s >= 4.5
                 AND Steam_Flow_tonne_per_h < 80
                 AND Boiler_Efficiency_pct < 88
              THEN 'Potentially Inefficient Operation'
        
            ELSE 'Normal'
          END AS operating_status
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        WHERE Fuel_Flow_kg_s IS NOT NULL
          AND Steam_Flow_tonne_per_h IS NOT NULL
          AND Boiler_Efficiency_pct IS NOT NULL;


  ### Create an overall anomaly score
  
        SELECT
          *,
          
          (
            IF(Boiler_Efficiency_pct < 87, 1, 0)
            +
            IF(Fuel_Flow_kg_s > 5, 1, 0)
            +
            IF(Flue_Gas_Temp_C > 220, 1, 0)
            +
            IF(Generator_Power_Factor < 0.85, 1, 0)
          ) AS anomaly_score
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`;
        
        
        WITH scored_data AS (
        
          SELECT
            *,
            
            (
              IF(Boiler_Efficiency_pct < 87, 1, 0)
              +
              IF(Fuel_Flow_kg_s > 5, 1, 0)
              +
              IF(Flue_Gas_Temp_C > 220, 1, 0)
              +
              IF(Generator_Power_Factor < 0.85, 1, 0)
            ) AS anomaly_score
        
          FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        )
        
        SELECT
          *,
          CASE
            WHEN anomaly_score = 0 THEN 'Normal'
            WHEN anomaly_score = 1 THEN 'Watch'
            WHEN anomaly_score >= 2 THEN 'Potential Anomaly'
          END AS anomaly_category
        
        FROM scored_data;


  ### High-efficiency operating conditions
  
  #### When boiler efficiency is high, what does the plant operating environment look like?
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow,
          AVG(Combustion_Air_Flow_km3h) AS avg_air_flow,
          AVG(Flue_Gas_Temp_C) AS avg_flue_gas_temp,
          AVG(Flue_Gas_O2_pct) AS avg_flue_gas_o2,
          AVG(Generator_Power_MW) AS avg_generator_power
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        WHERE Boiler_Efficiency_pct >= 92;

  ### compare high-efficiency vs low-efficiency operation.
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow,
          AVG(Combustion_Air_Flow_km3h) AS avg_air_flow,
          AVG(Flue_Gas_Temp_C) AS avg_flue_gas_temp,
          AVG(Flue_Gas_O2_pct) AS avg_flue_gas_o2,
          AVG(Generator_Power_MW) AS avg_generator_power
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        WHERE Boiler_Efficiency_pct < 88;

  ### What operating conditions are associated with high steam production?
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow,
          AVG(Boiler_Efficiency_pct) AS avg_boiler_efficiency,
          AVG(Turbine_Inlet_Pressure_bar) AS avg_turbine_pressure,
          AVG(Turbine_Inlet_Temp_C) AS avg_turbine_temperature,
          AVG(Generator_Power_MW) AS avg_generator_power
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        WHERE Steam_Flow_tonne_per_h >= 150;

  ### High generator-power conditions
  ### What conditions are associated with high electrical output?
  
        SELECT
          COUNT(*) AS observations,
        
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Turbine_Inlet_Pressure_bar) AS avg_turbine_pressure,
          AVG(Turbine_Inlet_Temp_C) AS avg_turbine_temperature,
          AVG(Turbine_Speed_rpm) AS avg_turbine_speed,
          AVG(Condenser_Vacuum_kPa) AS avg_condenser_vacuum,
          AVG(Generator_Power_Factor) AS avg_power_factor
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        WHERE Generator_Power_MW >= 80;

  ### Compare high vs low performance
        
        SELECT
          CASE
            WHEN Boiler_Efficiency_pct >= 92
                 AND Steam_Flow_tonne_per_h >= 130
                 AND Generator_Power_MW >= 70
              THEN 'High Performance'
        
            WHEN Boiler_Efficiency_pct < 88
                 OR Steam_Flow_tonne_per_h < 80
                 OR Generator_Power_MW < 40
              THEN 'Low Performance'
        
            ELSE 'Medium Performance'
          END AS performance_category,
        
          COUNT(*) AS observations,
        
          AVG(Steam_Flow_tonne_per_h) AS avg_steam_flow,
          AVG(Fuel_Flow_kg_s) AS avg_fuel_flow,
          AVG(Boiler_Efficiency_pct) AS avg_boiler_efficiency,
          AVG(Generator_Power_MW) AS avg_generator_power,
          AVG(Turbine_Inlet_Pressure_bar) AS avg_turbine_pressure,
          AVG(Turbine_Inlet_Temp_C) AS avg_turbine_temperature,
          AVG(Condenser_Vacuum_kPa) AS avg_condenser_vacuum
        
        FROM `boiler-turbine-plant-analytics.raw_data.boiler_turbine_sensor_data`
        
        GROUP BY performance_category
        ORDER BY
          CASE performance_category
            WHEN 'High Performance' THEN 1
            WHEN 'Medium Performance' THEN 2
            WHEN 'Low Performance' THEN 3
          END;
        SQL/14_business_insights.sql


## Dashboard
The completed dashboard is organized into four pages:

### 01 — Plant Overview
<img width="1526" height="1125" alt="image" src="https://github.com/user-attachments/assets/e106477b-9ea8-4799-9b6f-3622c92700a2" />


### 02 — Boiler Performance
<img width="1504" height="1125" alt="image" src="https://github.com/user-attachments/assets/439a7b03-b6e4-4f1d-8e1c-debc1df31edb" />


### 03 — Turbine & Generator
<img width="1496" height="1125" alt="image" src="https://github.com/user-attachments/assets/79b10753-2f51-408d-8027-2aa52445b9b0" />


### 04 — Operational Insights
<img width="1508" height="1125" alt="image" src="https://github.com/user-attachments/assets/2ddb7b89-ca5f-4153-9b6d-ccc817c53465" />


For scatter charts, the dashboard uses Average aggregation, about 1,000 bubbles and no descending sort so the displayed point distribution remains representative.

## Suggested portfolio bullet
> Built an end-to-end industrial plant performance analytics project using Google BigQuery, SQL and Data Studio, analyzing 100K+ boiler-turbine sensor records to evaluate boiler efficiency, steam generation, turbine/generator performance, condenser conditions and operational anomalies.


## Future enhancements
- Python EDA and statistical testing
- ML model to predict steam flow
- Feature importance
- Scheduled BigQuery tables/views
- Cloud Storage ingestion
- Automated data-quality checks
