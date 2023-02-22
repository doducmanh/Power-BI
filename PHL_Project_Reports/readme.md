<b>Summary:</b>
- We design, architecture, visualize reports on Power BI Desktop RS, create maesures by using DAX.
- Then upload, publish reports on to Power BI Report Server site, and schedule data refresh daily.

**Due to the company's privacy policy, I cannot share the data details and must to hide company logo*<p>
<b>Power BI Desktop:</b>
- Sale Performance Report:
![image](https://user-images.githubusercontent.com/59658937/220507247-53138d49-d262-4a11-921f-b3b7a17bd5ec.png)
![image](https://user-images.githubusercontent.com/59658937/220507328-3e3af334-7560-42ff-9c98-6618f322e0f8.png)

- CS Application - Premium:
![image](https://user-images.githubusercontent.com/59658937/220508113-2e1ad753-26d8-47c6-89ac-3117f1c83c92.png)
![image](https://user-images.githubusercontent.com/59658937/220508183-1e378a0d-fcf4-441b-a504-a028c93f7a55.png)

- Manpower Report:
![image](https://user-images.githubusercontent.com/59658937/220508740-1ce53512-fa4f-49c3-9b46-1b3015653d7b.png)
![image](https://user-images.githubusercontent.com/59658937/220508824-129e3795-98da-4ef1-9280-13a4f01939ae.png)

- K2 Report:
![image](https://user-images.githubusercontent.com/59658937/220512755-5e7911b9-c1f1-431d-8625-065f62ddab9c.png)

- MDRT/COT/TOT Report:
![image](https://user-images.githubusercontent.com/59658937/220513145-ec7bb708-bbbf-4844-8253-00fa5f2830fe.png)
![image](https://user-images.githubusercontent.com/59658937/220513187-bb1639f6-b164-444b-a089-bea0a32056c7.png)

- Hội An Trip Contest:
![image](https://user-images.githubusercontent.com/59658937/220505990-80fc08e5-0d2c-4d85-8fa4-a70c38fd0b75.png)
![image](https://user-images.githubusercontent.com/59658937/220506120-482872af-def5-41f8-8faa-8b6e2b2a3788.png)

<b>Power BI Report Server:</b>
![image](https://user-images.githubusercontent.com/59658937/220514755-040bcc72-8dec-45fd-b5a9-2e0a4fffae77.png)
![image](https://user-images.githubusercontent.com/59658937/220514667-9d5a556a-f097-4ba2-8efb-daf3eca0ed14.png)
-
<b>Some highlight DAX function:</b>
- Agent Name Lookup = 
Var Agent = IF(HASONEFILTER(Agency_Structure[Agent_Number]),values(Agency_Structure[Agent_Number]),BLANK())
RETURN LOOKUPVALUE(Agent_List[Agent_Name],Agent_List[Agent_Number], Agent)
  
- Agent_List - Service Month = 
VAR Agent = IF(HASONEFILTER(Agent_List[Agent_Number]), VALUES(Agent_List[Agent_Number]))
VAR AgentAppointed = LOOKUPVALUE(Agent_List[Date_Appointed],Agent_List[Agent_Number],Agent)
VAR fullMonthAppointed = IF(DAY(AgentAppointed)<=10,EDATE(AgentAppointed,-1),AgentAppointed)
VAR filterdate = MAX(DimDate[Date])
RETURN CALCULATE(DATEDIFF(IF(fullMonthAppointed>filterdate,filterdate,fullMonthAppointed),filterdate,MONTH))
  
- CC - AD = 
VAR AD = IF(HASONEVALUE(AD_List[AD_Code]),values(AD_List[AD_Code]),blank())
VAR L1 =
CALCULATE(
    [CC]
    ,FactADHistory[L1NP] = AD
)
VAR L2 =
CALCULATE(
    [CC]
    ,FactADHistory[L2NP] = AD
)
VAR L3 =
CALCULATE(
    [CC]
    ,FactADHistory[L3NP] = AD
)
VAR L4 =
CALCULATE(
    [CC]
    ,FactADHistory[L4NP] = AD
)
VAR L5 =
CALCULATE(
    [CC]
    ,FactADHistory[L5NP] = AD
)

RETURN L1+L2+L3+L4+L5
 
- CC_30Days 2 = 
VAR startdate = MIN(DimAgent[Date_Appointed])
VAR enddate = MAX(DimAgent[Date_Appointed])
RETURN
CALCULATE(SUM(Cal_AD_Daily_Policy[CountPolicy]),
ALLEXCEPT(Cal_AD_Daily_Policy,Cal_AD_Daily_Policy[Issuing Agent]),
Cal_AD_Daily_Policy[Date_Appointed] >= startdate && Cal_AD_Daily_Policy[Date_Appointed] <= enddate,
Cal_AD_Daily_Policy[DateDif] <=30 && Cal_AD_Daily_Policy[DateDif] >=0)
  
- FPL - Sale Team = 
VAR Agent = IF(HASONEFILTER(Agent_List[Agent_Number]),TOPN(1,VALUES(Agent_List[Agent_Number])),BLANK())
VAR AgentAppointedDate = LOOKUPVALUE(Agent_List[Date_Appointed],Agent_List[Agent_Number],Agent)
VAR AgentAppointedDateFullMonth =  IF(DAY(AgentAppointedDate) > 10, EDATE(AgentAppointedDate,-1),AgentAppointedDate)

VAR L1 =  CALCULATE(
    [Cal_FYPinclTopup],
    Agency_Structure[Leader]=Agent,
    Agency_Structure[GRADE] = "IC",
    Agency_Structure[Leader Grade]<>"IC",
    Cal_AD_Premium[Collected Date] >=AgentAppointedDateFullMonth
    )

VAR L2 =  CALCULATE(
    [Cal_FYPinclTopup],
    Agency_Structure[Agent_Number]=Agent,
    Agency_Structure[GRADE] <> "IC",
    Cal_AD_Premium[Collected Date] >=AgentAppointedDateFullMonth
    )
RETURN L1 + L2

- FPW - Case Size = 
VAR MinDate = Min(DimDate[Date])
VAR MaxDate = Max(DimDate[Date])
VAR MonthCount = DATEDIFF(MinDate,MaxDate,MONTH)+1
RETURN DIVIDE([Hoi An - Sale Personal],[FPW - CC Personal],0)

- FPW - FYP_no_Topup = 
VAR Agent = IF(HASONEFILTER(Agent_List[Agent_Number]),TOPN(1,VALUES(Agent_List[Agent_Number])),BLANK())
VAR L1 =  CALCULATE(
    [Cal_FYP_no_Topup],
    Agency_Structure[Leader]=Agent,
    Agency_Structure[GRADE] = "IC",
    Agency_Structure[Leader Grade] <> "IC"
    )

VAR SalePersonalAL =  CALCULATE(
    [Cal_FYP_no_Topup],
    Agency_Structure[Agent_Number]=Agent,
    Agency_Structure[GRADE] <> "IC"
    )
RETURN L1 + SalePersonalAL +0
  
- FYP - AD = 
VAR AD = IF(HASONEVALUE(AD_List[AD_Code]),values(AD_List[AD_Code]),blank())
VAR L1 =
CALCULATE(
    [Cal_FYPinclTopup]
    ,FactADHistory[L1NP] = AD
)
VAR L2 =
CALCULATE(
    [Cal_FYPinclTopup]
    ,FactADHistory[L2NP] = AD
)
VAR L3 =
CALCULATE(
    [Cal_FYPinclTopup]
    ,FactADHistory[L3NP] = AD
)
VAR L4 =
CALCULATE(
    [Cal_FYPinclTopup]
    ,FactADHistory[L4NP] = AD
)
VAR L5 =
CALCULATE(
    [Cal_FYPinclTopup]
    ,FactADHistory[L5NP] = AD
)

RETURN L1+L2+L3+L4+L5
  
