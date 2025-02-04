# healthcare
Data uploaded From Kaggle.
This dataset is synthetically generated and does not represent real patients.
It is meant for research and educational purposes only. 

# Review Data

Select *
From dbo.healthcare

# Creating a new table with updated column names, rounding, and data manipulation.

If OBJECT_ID('Dbo.Heartattack','U') IS NOT NULL
Drop Table Dbo.Heartattack
Go
Select 
		age as Age,
		Case when sex = 1 then 'Male' else 'Female' End as Gender,
		round(total_cholesterol,2) as Cholesterol,
		Round(ldl,2) as Low_Density_Lipoprotein,
		Round(hdl,2) as High_Density_Lipoprotein,
		Round(systolic_bp,2) as Systolic_bp,
		Round(diastolic_bp,2) as Diastolic_bp,
		Case when smoking = 1 then 'Yes' else 'No' End as Smoking,
		Case when diabetes = 1 then 'Yes' else 'No' end as Diabetes,
		Case when heart_attack = 1 then 'Yes' else 'No' end as Heart_Attack
		into Dbo.Heartattack
from dbo.healthcare

I created the new table dbo.Heartattack with all the modifications so that anyone who wants to see the data can read it easily.
The table can be updated because of the condition: IF OBJECT_ID('dbo.Heartattack', 'U') IS NOT NULL DROP TABLE dbo.Heartattack.

-- Review New Table

Select * 
from Dbo.Heartattack

# Divide the age group into three categories: <30, between 30 and 50, and >50. Then, find which group has the most heart attacks.
SELECT 
	Age_Group,
	COUNT(Heart_Attack) as Heart_Attack
From (
		Select 
				Heart_Attack,
				Case when Age < 30 Then 'under 30'
					 When Age Between 30 and 50 Then 'Between 30 & 50'
					 When Age > 50 Then 'Over 50'
					 End as Age_Group
		From Dbo.Heartattack
) A Group by Age_Group
	Order by COUNT(Heart_Attack) Desc

# Does smoking increase the risk of having a heart attack?

Select 
		Count(Heart_Attack) as Heart_Attack, 
		Smoking
From Dbo.Heartattack
Group By Smoking

# Do people with LDL cholesterol levels higher than 100 and HDL cholesterol levels lower than 60 
# Have a higher risk of heart attack? 

Select 
		count(Heart_Attack) as Affected,
		Heart_Attack
From Dbo.Heartattack
Where Low_Density_Lipoprotein > 100 and High_Density_Lipoprotein < 60 
group by Heart_Attack

# The level of detail provided by the window function will give more insight for each patient.
Select 
		count(*) over (partition by Heart_Attack) as Affected,
		Gender,
		Low_Density_Lipoprotein,
		High_Density_Lipoprotein,
		Heart_Attack
From Dbo.Heartattack
Where Low_Density_Lipoprotein > 100 and High_Density_Lipoprotein < 60 

# Do people with higher cholesterol have a greater probability of a heart attack? Provide the percentage.
Select *,
		sum(People_at_Risk) over () as TotalRisk,
		Round(cast(People_at_Risk as float) / sum(People_at_Risk) over () *100,2) as Total_Percentage
from(
Select count(Heart_Attack) as People_at_Risk,
		Categories
From(
	Select 
			Heart_Attack,
		Case when Cholesterol <200 then 'Normal'
			 when Cholesterol between 200 and 239 then 'Borderline high'
			 when Cholesterol >240 then 'High'
			 End as Categories
	From Dbo.Heartattack
)A 	Where Heart_Attack = 'Yes'
	group by Categories
) B

#  How many males over the age of 45 have diabetes?
Select Gender,
		COUNT(Diabetes) as Count_diabetes
From (
Select 
		Age,
		Gender,
		Diabetes	
from Dbo.Heartattack
Where age >45 and Gender = 'Male' and Diabetes = 'Yes'
) A group by Gender


# How many males over the age of 45 have diabetes and provide percentage?

Select 
		*,	
		sum(Count_) over () as Total,
		Round(CAST(Count_ AS float) /sum(Count_) over () *100,2) AS Percentages
From (
Select 
		Diabetes,
		Count(*) as Count_
from dbo.Heartattack	
Where age >45 and Gender = 'Male'
Group by Diabetes
) A


# How many Female over the age of 45 have not diabetes?

Select Gender,
		COUNT(Diabetes) as Count_diabetes
From (
Select 
		Age,
		Gender,
		Diabetes	
from Dbo.Heartattack
Where age >45 and Gender <> 'Male' and Diabetes <> 'Yes'
) A group by Gender










