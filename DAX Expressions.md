# Counts
```vbscript 
=
Distintcount([waarde])
```

# Running total
```vbscript 
=
IF(SELECTEDVALUE(Dim_Date[date]) >= TODAY(),BLANK(),
CALCULATE([Aantal Trajecten],
     FILTER(ALLSELECTED(Dim_Date[Date]),
     Dim_Date[Date] <= MAX(Dim_Date[Date])
     ),
     Dim_Date[Date] <= TODAY(),
     Dim_Date[Jaar] = 2024
))
```
# Regressie analyse<br>
```vbscript
=
VAR Known =
     FILTER (
         SELECTCOLUMNS (
             ALLSELECTED ( 'Dim_Date'[Date] ), // Date tabel date column
             "Known[X]", 'Dim_Date'[Date],
             "Known[Y]", [RunningTotalTrajecten] // Runing totals
         ),
         AND (
         NOT ( ISBLANK ( Known[X] ) ),
         NOT ( ISBLANK ( Known[Y] ) )
         )
     )
VAR SlopeIntercept =
     LINESTX(Known, Known[Y], Known[X])
VAR Slope =
     SELECTCOLUMNS(SlopeIntercept, [Slope1])
VAR Intercept =
     SELECTCOLUMNS(SlopeIntercept, [Intercept])
RETURN
     SUMX (
         DISTINCT ( 'Dim_Date'[Date] ),
         Intercept + Slope * 'Dim_Date'[Date]
     )
```
# Correlation Coefficient 
```vbscript
= 
VAR Correlation_Table =
    FILTER (
        ADDCOLUMNS (
            VALUES ( Data[Item] ),
            "Value_X", CALCULATE ( SUM ( Data[Value X] ) ),
            "Value_Y", CALCULATE ( SUM ( Data[Value Y] ) )
        ),
        AND (
            NOT ( ISBLANK ( [Value_X] ) ),
            NOT ( ISBLANK ( [Value_Y] ) )
        )
    )
VAR Count_Items =
    COUNTROWS ( Correlation_Table )
VAR Sum_X =
    SUMX ( Correlation_Table, [Value_X] )
VAR Sum_X2 =
    SUMX ( Correlation_Table, [Value_X] ^ 2 )
VAR Sum_Y =
    SUMX ( Correlation_Table, [Value_Y] )
VAR Sum_Y2 =
    SUMX ( Correlation_Table, [Value_Y] ^ 2 )
VAR Sum_XY =
    SUMX ( Correlation_Table, [Value_X] * [Value_Y] )
VAR Pearson_Numerator =
    Count_Items * Sum_XY - Sum_X * Sum_Y
VAR Pearson_Denominator_X =
    Count_Items * Sum_X2 - Sum_X ^ 2
VAR Pearson_Denominator_Y =
    Count_Items * Sum_Y2 - Sum_Y ^ 2
VAR Pearson_Denominator =
    SQRT ( Pearson_Denominator_X * Pearson_Denominator_Y )
RETURN
    DIVIDE ( Pearson_Numerator, Pearson_Denominator )
```
https://community.fabric.microsoft.com/t5/Quick-Measures-Gallery/Correlation-coefficient/m-p/196274

# Totals

```vbscript 
=
TOTALYTD()
```
