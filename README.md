This readme only applies to the version of the model without energy efficiency investment. The model is written in GAMS, and applies to these two papers:

1.)	Matar, Walid. “Households' response to changes in electricity pricing schemes: bridging microeconomic and engineering principles.” Energy Economics 75. 2018. 

2.) Matar, Walid. “Households’ Demand Response to Changes in Electricity Prices: A Microeconomic-Physical Approach.” KAPSARC Discussion Paper KS--2019-DP51. 2019. (forthcoming)

The main integration file (Main.gms) is presented in this text document. Comments in this readme file facilitate that user. All GAMS files are included in a zipped folder. Once the files are copied onto your local directory, the file Main.gms is the one that is run; the files will only run if they are all saved in the GAMS project directory that is opened. The project directory file “ResU.gpr” is included for convenience. All the other files appearing in the main file, like “Res_Sets.gms” and “utilitymax.gms”, are included in their original *.gms format. 

In addition, other files that contain data for the utility-maximization and residential electricity use components are referenced below the main integration file. 

The main changes over the version used by Matar (2018) are:
	Different electricity pricing scenarios are considered; one that includes output from an energy system model.

	Additional price response measures are included. Those measures affect more than just direct electricity use, but also the amount of heat gains by the indoor zone. A file is included to re-calculate the parameters associated with the heat gain.

	Since the energy efficiency measures in this analysis do not include thermal insulation of the envelope, references to insulation in the file set have been removed.

	The main.gms file has been streamlined for easier use. 










The main integration file
(Main.gms)
 
	Although we are not optimizing the residential model and running it with the same number of unknowns as we have equations, we use the solver PATHNLP.

option limrow=1e2, limcol=1e2,
NLP=PATHNLP,
       decimals=6, solprint=off, savepoint=0;

	The sets “count”, “count2”, “count3”, “countL”, and “countTV” are used by the model to cycle through indoor temperature set points, lighting use, and consumer electronics use. Each set is described below.

Sets
*Thermostat set points:
     count for summer thermostat set points /ci1*ci4/
     count_use(count) /ci1/
     count2 for spring-fall thermostat set points /ci1*ci3/
     count2_use(count2) /ci1/
     count3 for distinguishing peak hours in the summer /ci1*ci2/
     count3_use(count3) /ci1/
*Lights:
     countL for lightbulbs /ciL1*ciL3/
     countL_use(countL) /ciL1*ciL1/
*TV:
     countTV for "consumer electronics" /ciTV1*ciTV3/
     countTV_use(countTV) /ciTV1*ciTV1/

	The set “tarinc” defined the different electricity pricing schemes considered in the paper:
ti1: base case (progressive structure in place in 2017)
ti2: 2018 electricity pricing
ti3: TOU price
ti4: Real-time price

     tarinc tariff scenarios /ti1*ti4/
     tarinc_use(tarinc);

tarinc_use(tarinc)=yes;

	The main residential electricity use files, used to define sets, variables, equations, and data, are included below. The data file, “res_parameters.gms”, is further explored below.

$INCLUDE res_sets.gms
$INCLUDE res_parameters.gms

Parameter Elecshare(uses) share of electricity use by end-use;
Elecshare('cooling')=0.70;
Elecshare('lighting')=0.08;
Elecshare('conselec')=0.03;
Elecshare('other')=1-(Elecshare('cooling')+Elecshare('lighting')+Elecshare('conselec'));

	The file, “ResidentiallVilla.gms”, contains the equations that govern the electricity use component. 

$INCLUDE VarsEqsDeclaration.gms
$INCLUDE ResidentialVilla.gms
;
Parameter Tindoortariffinc,electariffinc,Utilityinc,utilitysave,tindoorsave,test          Utilityincount,Hourlyloadcount,elasticity,MonthlyelectricityconsumedTARINC1,          MonthlyelectricitycostTARINC1,illuminatemindisp,illuminatemin_ini,Tindoor_ini          LightingconsTAR,ACconsTAR,TVconsTAR,Lightingconscount,ACconscount,TVconscount          annualeleccons,avgunitprice,Hourlyloadcount,ACconscount,Lightingconscount,          TVconscount,annualeleccostcount,annualelecconscount,MonthlyelectricitycostTARINC1count,MonthlyelectricityconsumedTARINC1count,elasticitycount,avgunitpricecount,testcount,UTcountmax,UTcount2max,UTcount3max,UTcountLmax,UTcountTVmax,Hourlyloadtar,Illumtar,C1,C2,Q1,Q2,electariff(consbrack), electariffsave(consbrack) electricity price in SAR per kWh corresponding to each bracket,annualeleccost annual electricity expenditures in thousand USD,
RTPhour(hour,season,allregions) real-time prices in SAR per kWh;

electariff('b1')=5;
electariff('b2')=10;
electariff('b3')=20;
electariff('b4')=30;
electariff('b5')=30;
electariff('b6')=30;
electariff('b7')=30;
electariff('b8')=30;
electariff(consbrack)=electariff(consbrack)/100;
electariffsave(consbrack)=electariff(consbrack);

	Here, the real-time prices are imported using an Excel file from the KAPSARC Energy Model (KEM), an energy system model calibrated for Saudi Arabia.

$CALL GDXXRW.exe RTP_prices.xlsx par=RTP rng=RTP!a1:c25 Rdim=2 Cdim=1
Parameter RTP(ELl,season,allregions) real-time prices imported from KEM in USD per MWh;
$GDXIN RTP_prices.gdx
$LOAD RTP
$GDXIN

RTPhour(hour,season,allregions)$(ord(hour)<4)=RTP('L1',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=4 and ord(hour)<8)=RTP('L2',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=8 and ord(hour)<12)=RTP('L3',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=12 and ord(hour)<14)=RTP('L4',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=14 and ord(hour)<17)=RTP('L5',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=17 and ord(hour)<19)=RTP('L6',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=19 and ord(hour)<21)=RTP('L7',season,allregions);
RTPhour(hour,season,allregions)$(ord(hour)>=21)=RTP('L8',season,allregions);

*To convert $/MWh to SAR/kWh:
RTPhour(hour,season,allregions)=RTPhour(hour,season,allregions)*3.75/1e3;
display RTP, RTPhour;

Scalar elecfixedtariff fixed charge for meter reading in SAR per month /15/;

	The file, “utilitymax.gms”, contains the data and equations used to define the maximization of utility. It is further explored below. 

$INCLUDE utilitymax.gms

*These are just placeholders to be written over by the model:
utilitysave(count,count2,count3,countL,countTV)=1e2;
annualeleccons(tarinc)=1e-1;
ACconsTAR(tarinc)=1e-1;
LightingconsTAR(tarinc)=1e-1;
TVconsTAR(tarinc)=1e-1;

Loop(tarinc$(tarinc_use(tarinc)),

*this is just a placeholder
utilitysave(count)=1e2;

	This is where the different electricity pricing schemes in the analysis are set.

*Electricity price settings:
$INCLUDE elec_price_scenarios.gms

*Ensuring the base assumptions for lighting, indoor temperature, and consumer electronics usage are set:
illuminatemin=130;
illuminatemin_ini=illuminatemin;

Tindoor(hour,'Summ','Cent')=273.15+25;
Tindoor(hour,'Wint','Cent')=273.15+21;
Tindoor(hour,'Spfa','Cent')=273.15+22;
Tindoor_ini(hour,season,r_use)=Tindoor(hour,season,r_use);

equipmentschedule('conelec',season,daytype,hour)=0;
equipmentschedule('conelec',season,daytype,hour)$(ord(hour)>=21 and ord(hour)<24)=1;

	This file is used to initiate or reinitiate the internal heat gain parameters associated with the appliance and lighting use settings.

$INCLUDE redef_IHG.gms

Loop((count,count2,count3,countL,countTV),


*Indoor temperature, in Kelvin,
Tindoor(hour,'summ',r_use)=Tindoor_ini(hour,'summ',r_use)+ord(count)-1+(0.5$peak(hour))$(ord(count3)>1);
Tindoor(hour,'spfa',r_use)=Tindoor_ini(hour,'spfa',r_use)+1.5*(ord(count2)-1)/(card(count2)-1);

*The illumination requirement, in lumens/sq. m:
illuminatemin=illuminatemin_ini-40*(ord(countL)-1)/(card(countL)-1);

*The consumer electronics usage schedule, 1 for use in a given hour:
if( ord(countTV)=2, equipmentschedule('conelec',season,daytype,hour)$(ord(hour)>=21 and ord(hour)<24)=1;
                    equipmentschedule('conelec',season,daytype,hour)$(ord(hour)=23)=0;
  elseif ord(countTV)=3,
                    equipmentschedule('conelec',season,daytype,hour)$(ord(hour)>=21 and ord(hour)<24)=1;
                    equipmentschedule('conelec',season,daytype,hour)$(ord(hour)=23 or ord(hour)=22)=0;
  elseif ord(countTV)=4,
                    equipmentschedule('conelec',season,daytype,hour)=0;                    );

*Redefining the hourly internal heat gains (IHG) from lights and appliances:
$INCLUDE redef_IHG.gms

countscalar=0;
Repeat(Tindoor(hour,'summ',r_use)=Tindoor(hour,'summ',r_use)-0.5*countscalar;

$INCLUDE solve_residential.gms

*if((ord(tarinc)>=3), option solprint=on; );
display Tindoor,Monthlyelectricitycost,ELREprice,Monthlyelectricityconsumed;

if(        ord(tarinc)=1,  solve utilitymax using NLP maximizing utility;
    elseif ord(tarinc)>1,  solve utilitymax2 using NLP maximizing utility; );

option solprint=off;
countscalar=countscalar+1;
   until(   (utilitymax.modelstat=1 or utilitymax.modelstat=2)     )

*Repeat close
      );

$INCLUDE countpar.gms
;
*Count loop close
         );

$INCLUDE tarincpar.gms
;
*tariff scenario loop close
);

$INCLUDE Excel.gms
;






















































The file containing the computation of the household’s utility
(utilitymax.gms) 




The data pertaining to the utility function, which are fully described in the main body of the manuscript, are shown in the file, “utilitymax.gms”. This file also consists of the utility’s computational sets and equations. The electricity expenditures used in this file are found in the file “solve_residential.gms”, as highlighted below. The expenditures equation are written depending on the electricity price scenario run.


Monthlyelectricityconsumed is a parameter defined in the file, “solve_residential.gms”. After the electricity use component has solved, this parameter is computed as follows:

〖monthlyelectricityconsumed〗_season=∑_(hour,d)▒P_(season,d,hour)   〖DS〗_(season,d)/〖MS〗_season 

Where d is day type for weekends/holidays or weekdays, P_(season,d,hour) are the hourly power loads by season and day type, 〖DS〗_(season,d) are the numbers of days by day type in a season, and 〖MS〗_season are the numbers of months in a season. It has the units, kWh. 

Monthlyelectricitycost is also a parameter defined in the file, “solve_residential.gms”. After the electricity use component has solved, this parameter is computed by Equations 3 to 5 in the main paper. 















 
Scalar CDcoeff1,CDcoeff2,CDcoeff3,CDcoeff4,CDcoeff5,CDcoeff6 Cobb-Douglas coefficients (estimated)
       income the household's income in thousand USD per year
       countscalar used for solve statement
       stop used for solve statement
       elecpref preference share for electricity
;

elecpref=0.1025;

*AC
CDcoeff1=elecpref*Elecshare('cooling');
*Lighting
CDcoeff2=elecpref*Elecshare('lighting');
*TV
CDcoeff3=elecpref*Elecshare('conselec');
*Other electricity use
CDcoeff4=elecpref-(CDcoeff1+CDcoeff2+CDcoeff3);

*Other, non-electricity, consumption
CDcoeff5=1-elecpref;

income=10723*12/3.75/1e3*(1-0);


display CDcoeff1,CDcoeff2,CDcoeff3,CDcoeff4,CDcoeff5,income;

Variables
utility
;

Positive variables
othergoods consumption of other goods in thousand USD per year
savings    savings in thousand USD per year
ACcons     consumption of electricity by operating the AC
Lightingcons  consumption of electricity by operating lights
TVcons     consumption of electricity by operating the TV
Otherelec  consumption of electricity by operating other equipment
;

Equations
utilobj    the utility function to be maximized
budgeteq   monetary budget constraint in thousands of USD per year
ACconsbal  equation to compute ACcons
Lightingconsbal equation to compute Lightcons
TVconsbal  equation to compute TVcons
otherelecbal equation to compute other electricity use
otherelecbal2
;

*Initialization:
othergoods.l=1e0;
savings.l=1e0;
ACcons.l=1e0;
Lightingcons.l=1e0;
TVcons.l=1e0;
otherelec.l=1e0;

utilobj.. utility=e=
   (ACcons**CDcoeff1)
  *(Lightingcons**CDcoeff2)
  *(TVcons**CDcoeff3)
*Other electricity consumption:
  *(otherelec**CDcoeff4)
*Consumption of other goods:
  *(othergoods**CDcoeff5)
*Savings
*  *savings**CDcoeff6;
;

*Electricity consumption for AC, in MWh:
ACconsbal.. ACcons-sum((season,r_use,hour,daytype),
                       HVACpower(season,daytype,hour,r_use)*Daysinseason(season,daytype)*1
                       )/1e3=e=0;


*Electricity consumption for lighting, in MWh:
Lightingconsbal.. Lightingcons-sum((season,r_use,hour,daytype),
                  Lightingpower.l(season,daytype,hour,r_use)*Daysinseason(season,daytype)*1
                                   )/1e3=e=0;

*Electricity consumption for TV, in MWh:
TVconsbal.. TVcons-sum((season,r_use,hour,daytype),
                 equipsaturation('conelec')*equipmentschedule('conelec',season,daytype,hour)*equippowerrating('conelec',r_use)*Daysinseason(season,daytype)*1
                       )/1e3=e=0;

*Other electricity consumption, in MWh:
otherelecbal.. otherelec-(  sum((RHsup,Tsup,intake,season,r_use),Monthlyelectricityconsumed(RHsup,Tsup,intake,season,r_use)/1e3*Monthsinseason(season))
                           -(ACcons+Lightingcons+TVcons) )=e=0;

otherelecbal2.. otherelec-(    annualeleccons('ti1')
                             -(ACconsTAR('ti1')+LightingconsTAR('ti1')+TVconsTAR('ti1')) )=e=0;

;
budgeteq.. income
*Savings
*   -savings
*Expenditures on electricity:
   -sum((RHsup,Tsup,intake,season,r_use),Monthlyelectricitycost(RHsup,Tsup,intake,season,r_use)*Monthsinseason(season))/3.75/1e3
*Other expenditures
   -othergoods=e=0;
********************************************************************************
model utilitymax /utilobj,budgeteq,TVconsbal,Lightingconsbal,ACconsbal,otherelecbal/
;
model utilitymax2 /utilobj,budgeteq,TVconsbal,Lightingconsbal,ACconsbal,otherelecbal2/
;


 



The file containing the data for the electricity use component
(Res_Parameters.gms)



Data pertaining to residential electricity use are shown in the file, “Res_Parameters.gms”. The set for geographic regions of Saudi Arabia is allregions, as defined in “Res_Sets.gms”. For this analysis, the only active element of the regional set is Cent, for central. The individual parameters are also defined with descriptive text next to the parameter names in “Res_Parameters.gms”. 

The file is too long to show line-by-line, so below, we only highlight those data for weather conditions, solar radiation, and appliance use schedule. They also help show the in-line description of each parameter, which includes the units of measure. Other data may be explored in the file. Trigonometric relationships are used in the file to determine the solar irradiance incident on each external surface of the house using the solar DNI data.

The set season contains the elements,  Summ (summer), Wint (winter), and Spfa (spring and fall). Spring and fall are combined into one seasonal period to match the modeling rational of the KAPSARC Energy Model. The set equipment consists of individual appliances, like the refrigerator, the stove and oven, consumer electronics, the dish washer, and the clothes washing machine.  



 
Table Tout(hour,season,allregions) outdoor dry bulb temperature in Kelvin
+      Summ.Cent      Wint.Cent     Spfa.Cent
hr0        305.18         286.85        297.05
hr1        304.28         286.36        296.29
hr2        303.47         285.79        295.65
hr3        302.65         285.16        295.00
hr4        301.90         284.80        294.37
hr5        301.35         284.43        293.88
hr6        301.08         284.10        293.62
hr7        301.77         283.97        294.30
hr8        305.02         285.09        296.53
hr9        308.44         287.27        299.26
hr10       310.27         288.95        301.24
hr11       311.40         290.27        302.57
hr12       312.30         291.41        303.59
hr13       313.03         292.28        304.37
hr14       313.55         292.94        304.91
hr15       313.83         293.22        305.06
hr16       313.80         293.09        304.93
hr17       313.27         292.47        304.18
hr18       312.24         291.25        302.94
hr19       310.54         290.21        301.62
hr20       309.06         289.32        300.46
hr21       307.93         288.65        299.50
hr22       307.02         287.98        298.72
hr23       306.09         287.45        297.85
hr24                305.18                      286.85                   297.05

Table DNI(hour,season,allregions) direct normal irradiance in W per sq. m
+          Summ.Cent      Wint.Cent         Spfa.Cent
hr0        0.00           0.00              0.00
hr1        0.00           0.00              0.00
hr2        0.00           0.00              0.00
hr3        0.00           0.00              0.00
hr4        0.00           0.00              0.00
hr5        0.00           0.00              0.00
hr6        126.18         0.00              69.73
hr7        428.50         179.37            345.00
hr8        616.34         417.28            537.36
hr9        701.90         540.97            630.72
hr10       758.87         591.76            691.35
hr11       790.57         632.42            726.64
hr12       787.36         672.32            726.11
hr13       769.69         669.87            698.14
hr14       731.90         643.80            640.54
hr15       660.86         544.11            551.38
hr16       549.80         440.87            413.90
hr17       343.09         198.41            162.23
hr18       82.14          0.00              16.31
hr19       0.00           0.00              0.00
hr20       0.00           0.00              0.00
hr21       0.00           0.00              0.00
hr22       0.00           0.00              0.00
hr23       0.00           0.00              0.00
hr24       0.00           0.00              0.00

Table relativehumidityout(hour,season,allregions) outdoor relative humidity in fraction of 1
+          Summ.Cent        Wint.Cent      Spfa.Cent
hr0        0.20             0.62           0.40
hr1        0.21             0.64           0.41
hr2        0.22             0.66           0.42
hr3        0.23             0.69           0.44
hr4        0.24             0.70           0.45
hr5        0.25             0.72           0.47
hr6        0.26             0.74           0.48
hr7        0.27             0.75           0.48
hr8        0.25             0.72           0.45
hr9        0.20             0.64           0.39
hr10       0.17             0.58           0.35
hr11       0.16             0.53           0.31
hr12       0.15             0.49           0.29
hr13       0.14             0.47           0.28
hr14       0.14             0.44           0.27
hr15       0.13             0.43           0.27
hr16       0.13             0.43           0.27
hr17       0.13             0.45           0.28
hr18       0.14             0.47           0.29
hr19       0.15             0.51           0.31
hr20       0.16             0.53           0.33
hr21       0.17             0.56           0.35
hr22       0.18             0.58           0.36
hr23       0.19             0.60           0.38
hr24       0.20             0.62           0.40

Parameter equipmentschedule(equipment,season,daytype,hour) “1” denotes the use of equipment or appliance for a given hour throughout the day;
equipmentschedule(equipment,season,daytype,hour)=0;
equipmentschedule('fridge',season,daytype,hour)=1;
equipmentschedule('freezer',season,daytype,hour)=1;
equipmentschedule('waterpump',season,daytype,hour)$(ord(hour)=3)=1;
equipmentschedule('dishwasher',season,daytype,hour)$((ord(hour)>=21 and ord(hour)<25))=1;
equipmentschedule('washer',season,daytype,hour)$((ord(hour)>=17 and ord(hour)<23))=1;
equipmentschedule('dryer',season,daytype,hour)$((ord(hour)>=17 and ord(hour)<23))=1;
equipmentschedule('stoven',season,daytype,hour)$(ord(hour)=7 or ord(hour)=19)=1;
equipmentschedule('conelec',season,daytype,hour)$(ord(hour)>=21 and ord(hour)<24)=1;
equipmentschedule('otherapp',season,daytype,hour)$(ord(hour)>=12 and ord(hour)<23)=1;
equipmentschedule('waterheater',season,daytype,hour)$((ord(hour)>=5 and ord(hour)<9) or (ord(hour)>=17 and ord(hour)<22))=0.33*1;
equipmentschedule('waterheater','Wint',daytype,hour)$((ord(hour)>=5 and ord(hour)<9) or (ord(hour)>=17 and ord(hour)<22))=1;



