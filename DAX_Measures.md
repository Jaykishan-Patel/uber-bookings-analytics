
# 📘 DAX Measures – Uber Trip Analysis  

This document contains all the **DAX measures and calculations** used in the **Uber Trip Analysis Power BI Dashboard**, grouped by category.  

---

## 📅 Date & Calendar Measures  

### 📌 Calendar Table  
```DAX
Calender_Table = 
CALENDAR(MIN('Trip Details'[PickUp_Date]), MAX('Trip Details'[PickUp_Date]))
````

Creates a **date table** spanning from the earliest to the latest pickup date, enabling time-based analysis.

### 📌 Day Number

```DAX
Day num = WEEKDAY(Calender_Table[Date], 2)
```

Returns the **day of week (1 = Monday … 7 = Sunday)** for trend analysis.

### 📌 Weekday Name

```DAX
WeekDayName = FORMAT(Calender_Table[Date], "dddd")
```

Provides the **weekday name (Monday, Tuesday, etc.)** for clear visuals.

---

## 📊 Dynamic Measures & Titles

### 📌 Dynamic Measure Selector

```DAX
Dynamic Measure = {
    ("Total Booking", NAMEOF('Trip Details'[Total Booking]), 0),
    ("Total booking Value", NAMEOF('Trip Details'[Total booking Value]), 1),
    ("Total Distance", NAMEOF('Trip Details'[Total Distance]), 2)
}
```

Enables switching between KPIs (**Bookings, Revenue, Distance**) dynamically.

### 📌 Dynamic Title

```DAX
Dynamic Title =
IF('Dynamic Measure'[Measures Order] = 0, "Total Bookings",
IF('Dynamic Measure'[Measures Order] = 1, "Total Booking Value",
IF('Dynamic Measure'[Measures Order] = 2, "Total Trip Distance", "Other")))
```

Updates **visual titles dynamically** based on the selected KPI.

### 📌 Dynamic Titles for Time & Weekday Analysis

```DAX
Title for dynamic Measure = 
SELECTEDVALUE('Dynamic Measure'[Dynamic Title]) & " Pickup Time"

Title for dynamic Measure weekday = 
SELECTEDVALUE('Dynamic Measure'[Dynamic Title]) & " Pickup weekday"
```

Customizes chart titles for **time-based** and **weekday-based visuals**.

---

## 💵 Revenue & Fare Analysis

### 📌 Total Booking Value

```DAX
Total booking Value = 
SUMX('Trip Details', 'Trip Details'[fare_amount] + 'Trip Details'[Surge Fee])
```

Calculates the **total revenue** from fares and surge fees.

### 📌 Average Fare per Booking

```DAX
Average Fare Of Booking = 
DIVIDE([Total booking Value], [Total Booking], BLANK())
```

Provides **average revenue per trip**, useful for pricing analysis.

---

## 🚖 Trip Volume & Distance

### 📌 Total Bookings

```DAX
Total Booking = COUNT('Trip Details'[Trip ID])
```

Total number of trips.

### 📌 Total Trip Distance (Raw)

```DAX
[Total Distance] = SUM('Trip Details'[trip_distance])
```

Total distance travelled.

### 📌 Total Trip Distance (Formatted)

```DAX
Total Trip Distance = 
VAR total_distance = SUM('Trip Details'[trip_distance]) / 1000
RETURN CONCATENATE(FORMAT(total_distance, "0.0"), "K Miles")
```

Displays distance in **“K Miles”** format for readability.

### 📌 Average Trip Distance

```DAX
Average Trip Distance =
ROUND(DIVIDE(SUM('Trip Details'[trip_distance]), [Total Booking], BLANK()), 2)
```

Shows the **average distance per trip**.

---

## ⏱ Trip Duration

### 📌 Trip Duration (Minutes)

```DAX
Trip_Time = 
DATEDIFF('Trip Details'[Pickup Time], 'Trip Details'[Drop Off Time], MINUTE)
```

Calculates **trip duration in minutes**.

### 📌 Average Trip Time

```DAX
Average Trip Time = AVERAGE('Trip Details'[Trip_Time])
```

Provides the **average ride duration** across all trips.

---

## 🌙 Trip Classification

### 📌 Trip Type (Day/Night)

```DAX
Trip Type(Day/Night) = 
VAR btime = HOUR('Trip Details'[Pickup Time])
RETURN IF(btime >= 18 || btime < 6, "Night Trip", "Day Trip")
```

Classifies trips as **Day Trip** or **Night Trip** based on pickup hour.

---

## 📍 Location Analysis

### 📌 Farthest Trip

```DAX
Farthest Trip = 
VAR MaxDistance = MAX('Trip Details'[trip_distance])

VAR PickupLocation =
LOOKUPVALUE(
    'Location Table'[Location],
    'Location Table'[LocationID],
    CALCULATE(
        SELECTEDVALUE('Trip Details'[PULocationID]),
        'Trip Details'[trip_distance] = MaxDistance
    )
)

VAR DropoffLocation =
LOOKUPVALUE(
    'Location Table'[Location],
    'Location Table'[LocationID],
    CALCULATE(
        SELECTEDVALUE('Trip Details'[DOLocationID]),
        'Trip Details'[trip_distance] = MaxDistance
    )
)

RETURN
"Pickup: " & PickupLocation & " > Drop-off: " & DropoffLocation & 
" (" & FORMAT(MaxDistance, "0.0") & " miles)"
```

Finds the **longest trip** and displays its **pickup & drop-off locations**.

### 📌 Most Frequent Drop-off Point

```DAX
Most Frequent Dropoff Point =
VAR DropOffCounts =
    ADDCOLUMNS(
        SUMMARIZE('Trip Details', 'Location Table'[Location]),
        "DropOffCount",
        CALCULATE(
            COUNT('Trip Details'[Trip ID]),
            USERELATIONSHIP('Trip Details'[DOLocationID], 'Location Table'[LocationID])
        )
    )

VAR RankedDropoffs =
    ADDCOLUMNS(DropOffCounts, "Rank", RANKX(DropOffCounts, [DropOffCount],, DESC, DENSE))

VAR TopDropoff = FILTER(RankedDropoffs, [Rank] = 1)

RETURN CONCATENATEX(TopDropoff, 'Location Table'[Location], ", ")
```

Finds the **most common drop-off location** using an inactive relationship.

---

## ⏰ Pickup Time Details

### 📌 Pickup Hour

```DAX
Pickup Hour = HOUR('Trip Details'[Pickup Time])
```

Extracts the **hour (0–23)** from pickup time.

### 📌 Pickup Time (Linked Table)

```DAX
Pickup Time = RELATED('Trip Details (2)'[Pickup Time])
```

Brings **pickup time** from a related table for analysis.

### 📌 Pickup Time (hh\:mm\:ss)

```DAX
Pickup Time(dd:mm:ss) = 
TIME(HOUR('Trip Details'[Pickup Time]), MINUTE('Trip Details'[Pickup Time]), SECOND('Trip Details'[Pickup Time]))
```

Formats pickup time in **hh\:mm\:ss** format.

### 📌 Pickup Date

```DAX
PickUp_Date = 
DATE(YEAR('Trip Details'[Pickup Time]), MONTH('Trip Details'[Pickup Time]), DAY('Trip Details'[Pickup Time]))
```

Extracts the **date only** (ignores time component).


