# Air Quality Prediction System - Grade A Extension

## Overview
For Grade A, I extended the system to predict air quality for **multiple sensors** in the same city. Instead of one location, the model now handles **5 different sensors** across Tampere, Finland.

## Multi-Sensor Setup

### Selected City: Tampere, Finland

### Sensors Used

| Sensor Name | Location | API ID |
|-------------|----------|--------|
| tampere | City center | H5719 |
| kaleva | Kaleva district | H4919 |
| pirkankatu | Pirkankatu street | H4921 |
| epila-2 | Epilä area | H4918 |
| linja-autoasema | Bus station | H4920 |

**Total: 5 sensors** tracking PM2.5 levels across different parts of the city.

## System Architecture

### 1. Data Collection
- **One weather forecast** for the whole city (same for all sensors)
- **Five air quality sensors** with different readings at different locations
- Each sensor has its own historical data

### 2. Feature Engineering
Each sensor has:
- Weather features (shared across all sensors)
- Lagged features (unique to each sensor)
- Location identifier (`street` column)

### 3. Model Training Strategy

I used **one unified model** for all sensors:
- Input: weather + lag features + **sensor location** (encoded as categorical feature)
- Output: PM2.5 prediction for that specific sensor
- This approach is better than 5 separate models because:
  - Learns patterns across all sensors
  - Easier to maintain
  - Can share knowledge between similar locations

### 4. Prediction Pipeline

**Daily Process:**
```
1. Get weather forecast for Tampere (7 days)
2. For each sensor:
   a. Get last 3 days PM2.5 values
   b. Add lag features
   c. Predict next 7 days
3. Generate 5 forecast charts
4. Generate 5 hindcast charts (prediction vs actual)
```

## Results

### Model Performance
- **MSE**: 50.73
- **R² Score**: 0.60

This is good performance considering:
- Tampere has very clean air (PM2.5 often < 10)
- Small values = harder to predict accurately
- Single model handles 5 different locations

### Prediction Examples

Each sensor gets its own forecast:
- `pm25_forecast_tampere.png`
- `pm25_forecast_kaleva.png`
- `pm25_forecast_pirkankatu.png`
- `pm25_forecast_epila-2.png`
- `pm25_forecast_linja-autoasema.png`

Plus hindcast charts showing prediction accuracy:
- `pm25_hindcast_tampere.png`
- `pm25_hindcast_kaleva.png`
- ... (5 charts total)

**Total: 10 charts** updated daily

## Implementation Details

### Backfill Pipeline
```python
sensors = [
    {"street": "tampere", "api_url": "..."},
    {"street": "kaleva", "api_url": "..."},
    # ... 5 sensors total
]

# Read all CSV files
for sensor in sensors:
    df = pd.read_csv(f"data/{sensor['street']}_air_quality.csv")
    # Add lag features per sensor
    # Combine all data
```

### Daily Feature Pipeline
```python
# Get weather once (same for all sensors)
weather = get_weather_forecast(city)

# Get air quality for each sensor
for sensor in sensors:
    aq = get_pm25(sensor['api_url'])
    # Add lag features based on sensor's history
    insert_to_feature_group(aq)
```

### Training Pipeline
```python
# One model for all sensors
features = ['pm25_lag_1', 'pm25_lag_2', 'pm25_lag_3', 
            'street',  # <-- This distinguishes sensors
            'temperature', 'wind', 'precipitation']

model = XGBRegressor()
model.fit(features, labels)
```

### Batch Inference Pipeline
```python
# Predict for each sensor separately
for sensor in sensors:
    # Get sensor's recent PM2.5 values
    # Predict recursively (use predictions as new lags)
    # Save forecast chart
```


## Automation

The system runs automatically every day at 06:11 UTC:
1. Collect yesterday's air quality from all 5 sensors
2. Get 7-day weather forecast
3. Generate predictions for all sensors
4. Create 10 charts (5 forecasts + 5 hindcasts)
5. Upload to GitHub Pages
