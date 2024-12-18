(*Import the data from an Excel file*)(*Import the data from an Excel \
file*)filePath = 
  "/...csv";
allEntries = Flatten[Import[filePath]];
priceSeries = allEntries[[-8760 ;;]];
Print["series length is", Length[priceSeries]]
(*Ensure the data is in the correct format*)
If[Not[VectorQ[priceSeries, NumericQ]], Print["Wrong format."];
  Return[$Failed];];

(*Categorize each price into'High','Medium',or'Low' based on \
quantiles*)quantiles = Quantile[priceSeries, {1/3, 2/3}];
priceCategories = 
  Map[If[# <= quantiles[[1]], "Low", 
     If[# <= quantiles[[2]], "Medium", "High"]] &, priceSeries];

colors = 
  priceCategories /. {"Low" -> Blue, "Medium" -> Orange, 
    "High" -> Red};

ListPlot[
 Style[{#1, #2}, #3] & @@@ 
  Transpose[{Range[Length[priceSeries]], priceSeries, colors}], 
 PlotStyle -> PointSize[Large], AxesLabel -> {"Index", "Price"}, 
 GridLines -> Automatic, PlotRange -> All, 
 PlotLabel -> "Scatter Plot of Price Series Categorized by Quantiles"]

(*Map categories to numeric values:'Low'->1,'Medium'->2,'High'->3*)
numericPriceCategories = 
  Map[Switch[#, "Low", 1, "Medium", 2, "High", 3] &, priceCategories];


(*Ensure numericPriceCategories is the same length as priceSeries*)
If[Length[numericPriceCategories] != Length[priceSeries], 
  Print["Error: Length mismatch."]; Return[$Failed];];
(*Define the states of the battery*)
states = {"Charging", "Discharging", "Idle"};

(*Function to calculate transition frequencies*)
calculateTransitionFrequencies[priceCategories_List] := 
  Module[{transitions, transitionMatrix, 
    n},(*Count transitions between states*)
   transitions = Tally[Partition[priceCategories, 2, 1]];
   (*Initialize transition matrix*)
   transitionMatrix = ConstantArray[0, {3, 3}];
   (*Fill the transition matrix with frequencies*)
   Do[With[{from = transition[[1, 1]], to = transition[[1, 2]], 
      count = transition[[2]]}, 
     transitionMatrix[[from, to]] += count], {transition, 
     transitions}];
   (*Normalize the rows to get probabilities*)
   n = Total[transitionMatrix, {2}];
   transitionMatrix = DiagonalMatrix[1/n] . transitionMatrix;
   transitionMatrix];

(*Calculate transition frequencies for numericPriceCategories*)
transitionFrequencies = 
  calculateTransitionFrequencies[numericPriceCategories];
Print[Grid[{{"Transition Frequencies Matrix:"}, {MatrixForm[
     transitionFrequencies]}}]]

(*Define prior probabilities*)
priorEmissionMatrix = {{0.5, 0.25, 0.25}, {0.25, 0.25, 0.5}, {0.25, 
    0.5, 0.25}};

(*Function to update emission probabilities using Bayesian approach \
with pseudo-Laplace smoothing in order not have the matrix filled \
with zeros*)
updateEmissionProbabilities[priorMatrix_, priceCategories_, 
   simulatedData_, smoothingFactor_ : 0.01] := 
  Module[{categoryStatePairs, categoryCounts, updatedMatrix, 
    totalPerCategory, 
    normalizedMatrix},(*Pair each price category with the \
corresponding simulated state*)
   categoryStatePairs = Transpose[{priceCategories, simulatedData}];
   (*Count occurrences of each state for each price category with \
smoothing*)
   categoryCounts = 
    Table[Count[categoryStatePairs, {category, state}] + 
      smoothingFactor, {category, {1, 2, 3}}, {state, {1, 2, 3}}];
   Print["Category Counts: ", 
    categoryCounts];(*Diagnostic print*)(*Update the emission matrix*)
   updatedMatrix = 
    MapThread[(#1*#2) &, {priorMatrix, categoryCounts}, 2];
   Print["Updated Matrix before Normalization: ", 
    updatedMatrix];(*Diagnostic print*)(*Normalize the updated matrix*)
   normalizedMatrix = Normalize[#, Total] & /@ updatedMatrix;
   Print["Normalized Matrix: ", normalizedMatrix];(*Diagnostic print*)
   normalizedMatrix];


(*Function to simulate battery states based on price categories and \
emission probabilities*)
simulateBatteryStatesFromPrices[priceCategories_, emissionMatrix_] := 
  Module[{simulatedStates}, 
   simulatedStates = 
    RandomChoice[emissionMatrix[[#]] -> {1, 2, 3}] & /@ 
     priceCategories;
   simulatedStates];

(*Simulate battery states for the first 2190 hours using the renamed \
function*)
simulatedBatteryStates = 
  simulateBatteryStatesFromPrices[numericPriceCategories[[1 ;; 2190]],
    priorEmissionMatrix];


(*Update the emission matrix based on the simulation of the first \
quarter*)
updatedEmissionMatrix = 
  updateEmissionProbabilities[priorEmissionMatrix, 
   numericPriceCategories[[1 ;; 2190]], simulatedBatteryStates];
Print[Grid[{{"Emission matrix: "}, {MatrixForm[
      updatedEmissionMatrix]}}]];

(*Function to calculate initial state probabilities*)
calculateInitialStateProbabilities[priceCategories_List] := 
  Module[{counts, total},(*Count occurrences of each category*)
   counts = Counts[priceCategories];
   (*Total number of prices*)total = Length[priceCategories];
   (*Calculate probabilities*){counts["Low"]/total, 
    counts["Medium"]/total, counts["High"]/total}];

(*Calculate initial state probabilities for priceCategories*)
initialStateProbabilities = 
  calculateInitialStateProbabilities[priceCategories];
Print[Grid[{{"Normalized Initial State Probabilities:"}, \
{initialStateProbabilities}}]]
(*Count and print the number of cycles*)


(*Create the Hidden Markov Model*)
HMM = HiddenMarkovProcess[initialStateProbabilities, 
   transitionFrequencies, updatedEmissionMatrix];


(*Find the most likely sequence of states*)
viterbiPath = 
  FindHiddenMarkovStates[numericPriceCategories, HMM, 
   "ViterbiDecoding"];


(*Check if viterbiPath is a valid list*)
If[Not[ListQ[viterbiPath]], 
  Print["Error: FindHiddenMarkovStates did not return a valid \
list."];
  Return[$Failed];];



(*Path Length*)pathLength = Length[viterbiPath];

(*Ensure lengths of priceSeries and viterbiPath match*)
If[Length[priceSeries] != Length[viterbiPath], 
  Print["Error: Length mismatch between priceSeries and \
viterbiPath"];
  Return[$Failed];];
(*Define Parameters and Variables*)
variables = 
  Flatten[Table[{pCharge[t], pDischarge[t], soc[t]}, {t, 1, 
     pathLength}]];

(*Parameters for 1 MW*)
pMax = 1; (*Maximum charge/discharge power in MW*)
socMin = 0.8; (*Minimum SoC in MWh*)
socMax = 4; (*Maximum SoC in MWh*)
eta = 0.85; (*Round-trip efficiency*)
soc0 = 0.8; (*Initial SoC*)
penaltyWeight = 10^7; (*Penalty for diverging from ViterbiPath*)
lambdaUtil = 10^10; (*Weight for utilization rate*)
maxYearlyCapacity = pMax*pathLength; (*Max yearly capacity in kWh*)

(*Initialize Objective and Constraints*)
objective = 0;
numericConstraints = {};
AppendTo[numericConstraints, soc[1] == soc0];

(*Iterate Through the Path*)
For[t = 1, t <= pathLength, t++,(*State Constraints and Penalty*)
  Which[viterbiPath[[t]] == 1,(*Charging State*)
   AppendTo[numericConstraints, 
    0 <= pCharge[t] <= pMax && pDischarge[t] == 0];
   penalty = 
    penaltyWeight*Abs[pDischarge[t]];(*Penalty for discharging*)
   objective -= (pCharge[t]*priceSeries[[t]]) + penalty;, 
   viterbiPath[[t]] == 2,(*Discharging State*)
   AppendTo[numericConstraints, 
    0 <= pDischarge[t] <= pMax && pCharge[t] == 0];
   penalty = penaltyWeight*Abs[pCharge[t]];(*Penalty for charging*)
   objective += (pDischarge[t]*priceSeries[[t]]) - penalty;
   objective += 
    lambdaUtil*(pDischarge[t]/
       maxYearlyCapacity); (*Add utilization term*), 
   viterbiPath[[t]] == 3,(*Idle State*)
   AppendTo[numericConstraints, pCharge[t] == 0 && pDischarge[t] == 0];
   penalty = 
    penaltyWeight*(Abs[pCharge[t]] + 
       Abs[pDischarge[t]]);(*Penalty for any action*)
   objective -= penalty;];
  (*SoC Bounds*)
  AppendTo[numericConstraints, socMin <= soc[t] <= socMax];
  (*SoC Continuity*)
  If[t < pathLength, 
   AppendTo[numericConstraints, 
     soc[t + 1] == soc[t] + eta*pCharge[t] - pDischarge[t]/eta];];];



(*Combine Constraints*)
constraints = And @@ numericConstraints;


(*Optimize*)result = NMaximize[{objective, constraints}, variables];
result



(*Define start time for timestamps*)
startDateTime = DateObject[{2023, 11, 01, 0, 0, 0}];

(*Generate timestamps with correct formatting*)
timestamps = 
  Table[DateString[
    DatePlus[startDateTime, {t - 1, "Hour"}], {"Year", "-", "Month", 
     "-", "Day", "T", "Hour", ":00"}], {t, 1, pathLength}];


(*Extract Optimized Values from the Result*)
socValues = Table[soc[t] /. Last[result], {t, 1, pathLength}];
pChargeValues = 
  Table[pCharge[t] /. Last[result], {t, 1, pathLength}];
pDischargeValues = 
  Table[pDischarge[t] /. Last[result], {t, 1, pathLength}];

(*Hourly gain calculation:Revenue from discharging minus cost of \
charging*)
hourlyGain = 
  Table[pDischargeValues[[t]]*priceSeries[[t]] - 
    pChargeValues[[t]]*priceSeries[[t]], {t, 1, pathLength}];

(*Combine all data into a single table*)
hourlyData = 
  Prepend[Table[{timestamps[[t]], socValues[[t]], priceSeries[[t]], 
     pChargeValues[[t]], pDischargeValues[[t]], hourlyGain[[t]], 
     viterbiPath[[t]] (*State extracted from viterbiPath*)}, {t, 1, 
     pathLength}], {"Timestamp", "SoC", "Price", "Charging Power", 
    "Discharging Power", "Hourly Gain", "State"}];

(*Monthly Summary:Group data by month*)
monthGrouping = 
  GroupBy[Table[{DateString[
      DateObject[timestamps[[t]]], {"Year", "Month"}],(*Month key*)
     hourlyGain[[t]],(*Gain*)
     pChargeValues[[t]]*priceSeries[[t]],(*Charging cost*)
     pDischargeValues[[t]]*priceSeries[[t]],(*Discharging revenue*)
     pChargeValues[[t]],(*Charged volume*)
     pDischargeValues[[t]] (*Discharged volume*)}, {t, 1, 
     pathLength}], First (*Group by the month key*)];

(*Compute monthly summaries*)
monthlySummary = 
  Prepend[Table[
    With[{data = monthGrouping[month]}, {month, 
      Total[data[[All, 2]]],(*Total Gain*)
      Total[data[[All, 3]]/
         Total[data[[All, 5]]] /. _ /; Total[data[[All, 5]]] == 0 -> 
         Missing[]],(*Captured Charging Price*)
      Total[data[[All, 4]]/
         Total[data[[All, 6]]] /. _ /; Total[data[[All, 6]]] == 0 -> 
         Missing[]]  (*Captured Discharging Price*)}], {month, 
     Keys[monthGrouping]}], {"Month", "Total Gain", 
    "Captured Charging Price", "Captured Discharging Price"}];

(*Export to Excel*)
Export["/Users/erykklossowski/Documents/Invite Energy/Gniewny Gieniek \
Gienerator/Battery_Optimization_Report.xlsx", {"Hourly Data" -> 
    hourlyData, "Monthly Summary" -> monthlySummary}];
Print["Report has been exported to Battery_Optimization_Report.xlsx"];

(*Plot Hourly Gain*)ListLinePlot[hourlyGain, PlotRange -> All, 
 AxesLabel -> {"Time", "Hourly Gain"}, 
 PlotLabel -> "Hourly Gain Over Time", ImageSize -> Large]

(*Plot State of Charge (SoC)*)
ListLinePlot[socValues, PlotRange -> All, 
 AxesLabel -> {"Time", "State of Charge (SoC)"}, 
 PlotLabel -> "State of Charge Over Time", ImageSize -> Large]

(*Plot Price Series*)
ListLinePlot[priceSeries, PlotRange -> All, 
 AxesLabel -> {"Time", "Price"}, 
 PlotLabel -> "Price Series Over Time", ImageSize -> Large]


(*Plot Charging and Discharging Power with SoC*)ListLinePlot[{{#, 
     pChargeValues[[#]]} & /@ 
   Range[Length[pChargeValues]], {#, pDischargeValues[[#]]} & /@ 
   Range[Length[pDischargeValues]], {#, socValues[[#]]} & /@ 
   Range[Length[socValues]]}, 
 PlotLegends -> {"Charging Power", "Discharging Power", "SoC"}, 
 PlotStyle -> {Blue, Red, Green}, PlotRange -> All, 
 AxesLabel -> {"Time", "Power/SoC"}, 
 PlotLabel -> "Charging, Discharging Power, and SoC", 
 ImageSize -> Large]

(*Extract Monthly Data*)
monthlyTotalGain = monthlySummary[[2 ;;, 2]]; (*Total Gain*)
capturedChargingPrices = 
  monthlySummary[[2 ;;, 3]]; (*Captured Charging Price*)
capturedDischargingPrices = 
  monthlySummary[[2 ;;, 4]]; (*Captured Discharging Price*)

(*Bar Chart for Monthly Total Gain*)
BarChart[monthlyTotalGain, ChartLabels -> monthlySummary[[2 ;;, 1]], 
 PlotLabel -> "Monthly Total Gain", 
 AxesLabel -> {"Month", "Total Gain (€)"}, ImageSize -> Large]

(*Line Plot for Captured Charging/Discharging Prices*)
ListLinePlot[{capturedChargingPrices, capturedDischargingPrices}, 
 PlotLegends -> {"Captured Charging Price", 
   "Captured Discharging Price"}, PlotStyle -> {Blue, Orange}, 
 PlotRange -> All, AxesLabel -> {"Month", "Captured Price (€)"}, 
 PlotLabel -> "Captured Charging and Discharging Prices", 
 ImageSize -> Large]
(*Scatter Plot of SoC vs Price*)ListPlot[
 Transpose[{socValues, priceSeries}], AxesLabel -> {"SoC", "Price"}, 
 PlotLabel -> "State of Charge vs Price", ImageSize -> Large, 
 PlotStyle -> Red]
