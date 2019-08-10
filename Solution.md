# Question 1
### In a tumbling window of 1 minute count the number of Audis that passed through a toll station.
SELECT <br />
count([input1].[vehicletypeid]), [cardata].[car_make],[input1].[spottype] <br />
<br />
FROM <br />
[input1]join[cardata]<br />
<br />
ON [input1].[vehicletypeid]=cast([cardata].[id] as bigint)<br />
<br />
WHERE cast([input1].[spottype] as nvarchar(max)) = 'Toll_Station'<br />
<br />
AND CAST([cardata].[CAR_MAKE] AS NVARCHAR(MAX)) = 'Audi'<br />
<br />
Group by [cardata].[car_make],[input1].[spottype],TumblingWindow (second,60)<br />
Result (Q1)

# Question 2
### In a hopping window of 3 minutes, for each color, calculate the total number of cars that passed through a police speed limit camera. Repeat every 90 seconds. 
SELECT <br />
count([colors].[color_name]),[colors].[color_name],[input1].[spottype], System.Timestamp As Time <br />
into [output]<br />
<br />
FROM<br />
[input1]join[colors]<br />
<br />
ON [input1].[colorid]=cast([colors].[color_code] as bigint)<br />
<br />
WHERE cast([input1].[spottype] as nvarchar(max)) = 'Speed_Limit_Camera'<br />
<br />
Group by [colors].[color_name],[input1].[spottype], HoppingWindow(second,180,90) <br />

Result(Q2)

# Question 3
### In a tumbling window of 20 seconds, for each color, find the oldest car that passed through a toll station.
SELECT <br />
[INPUT1].[COLORID], MIN([CARDATA].[CAR_MODEL_YEAR] )<br />
<br />
FROM [INPUT1]<br />
<br />
INNER JOIN [CARDATA]
ON [INPUT1].[VEHICLETYPEID] =[CARDATA].[ID]<br />
<br />
GROUP BY [INPUT1].[COLORID], TUMBLINGWINDOW(SECOND,90)<br />
<br />
Result(Q3)

# Question 4
### In a sliding window of 60 seconds, calculate the speed limit camera spots where the most violations happened. 
SELECT <br /> 
COUNT([SPEEDCAMERASPOTS].[ID]),[SPEEDCAMERASPOTS].[CITY], <br />
MAX([INPUT1].[SPEED]),[SPEEDCAMERASPOTS].[SPEED_LIMIT] <br />
<br />
FROM [INPUT1] <br />
INNER JOIN [SPEEDCAMERASPOTS] <br />
ON [SPEEDCAMERASPOTS].[ID]=[INPUT1].[CHECKPOINTID]<br />
<br />
WHERE CAST([INPUT1].[SPEED] AS BIGINT)> CAST([SPEEDCAMERASPOTS].[SPEED_LIMIT]
AS BIGINT)<br />
<br />
GROUP BY [SPEEDCAMERASPOTS].[CITY],[INPUT1].[SPEED],
[SPEEDCAMERASPOTS].[SPEED_LIMIT],SLIDINGWINDOW(SECOND,60)<br />

Result(Q4)

# Question 5
### In a sliding window of five minutes, for each color and car model, display the total number of cars that break the speed limit.
SELECT <br />
COUNT([INPUT1].[VEHICLETYPEID]),[COLORS].[COLOR_NAME],[CARDATA].[CAR_MODEL],[I
NPUT1].[SPEED],[SPEEDCAMERASPOTS].[SPEED_LIMIT]<br />
<br />
FROM ((([INPUT1]
INNER JOIN [COLORS] ON [INPUT1].[COLORID]= [COLORS].[COLOR_CODE]
INNER JOIN [CARDATA] ON [INPUT1].[VEHICLETYPEID] =[CARDATA].[ID]
INNER JOIN [SPEEDCAMERASPOTS] ON
[INPUT1].[CHECKPOINTID]=[SPEEDCAMERASPOTS].[ID])))<br />
<br />
WHERE CAST([INPUT1].[SPEED] AS BIGINT)>CAST([SPEEDCAMERASPOTS].[SPEED_LIMIT]
AS BIGINT)<br />
<br />
GROUP BY
[COLORS].[COLOR_NAME],[CARDATA].[CAR_MODEL],[INPUT1].[SPEED],[SPEEDCAMERASPOTS
].[SPEED_LIMIT], SLIDINGWINDOW(MINUTE,5)<br />

Result(Q5)
# Question 6
### You have been given a list of the license plates of police’s most wanted criminals. In a sliding window of 1 minute, display a list of all the cars that you spotted at any checkpoint.
SELECT <br />
[input1].[licensePlate],
count([input1].[licensePlate]) OVER(PARTITION BY [WantedCars].[WantedCars]) as
Wanted
into [output] <br />
<br />
FROM
[input1]join[WantedCars]<br />
ON
[input1].[licensePlate]=[WantedCars].[WantedCars]<br />
<br />
Group by[input1].[licensePlate], SLIDINGWINDOW(MINUTE,1)

# Question 7
### In a sliding window of 1 minute, display a list of fake license plates. Check if the same license plate has passed through any type of checkpoint twice in the same time window.
SELECT <br />
licensePlate,
count(licensePlate) OVER(PARTITION BY licensePlate) as Duplication
into [output]<br />
<br />
FROM
[input1] <br />
Group by licensePlate, SLIDINGWINDOW(MINUTE,1)
HAVING<br />
Duplication>1

# Question 8
### In a tumbling window of 2 minutes, calculate the percentage of BMW drivers that break the speed limit. (eg Out of all the B
MW drivers that were identified in the last 2 minutes, 80% broke the speed limit).

WITH TotalBMW AS (
SELECT
[INPUT1].[VEHICLETYPEID],[INPUT1].[SPEED],[SPEEDCAMERASPOTS].[SPEED_LIMIT] <br />
FROM <br />
([INPUT] <br />
INNER JOIN [CARDATA] ON [INPUT1].[VEHICLETYPEID] =[CARDATA].[ID]<br />
INNER JOIN [SPEEDCAMERASPOTS] ON<br />
[INPUT1].[CHECKPOINTID]=[SPEEDCAMERASPOTS].[ID])<br />
<br />
WHERE
CAST([cardata].[CAR_MAKE] AS NVARCHAR(MAX)) = 'BMW'
) <br />
SELECT <br />
CASE<br />
WHEN<br />
CAST([TotalBMW].[SPEED] AS BIGINT)>CAST([TotalBMW].[SPEED_LIMIT] AS BIGINT) <br />
THEN COUNT([TotalBMW].[VEHICLETYPEID])<br />
ELSE NULL<br />
END AS Fast <br />
FROM<br />
TotalBMW<br />
<br />
GROUP BY [TotalBMW].[SPEED],[TotalBMW].[SPEED_LIMIT],
TUMBLINGWINDOW(SECOND,120)
