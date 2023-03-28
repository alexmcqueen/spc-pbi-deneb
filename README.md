# Statistical Process Control (SPC) NHS Deneb Custom Visual for Power BI
Code relating to the creation of a XMR Statistical Process Charts (SPC) in a UK NHS setting using vega-lite in the Deneb custom visualisation. 

![SPC Chart Example](docs/images/SPC%20Chart%20Example%201.PNG)

## SPC NHS Overview

SPC is an analytical technique that plots data over time. It helps us understand variation and in so doing guides us to take the most appropriate action.

SPC is a good technique to use when implementing change as it enables you to understand whether changes you are making are resulting in improvement — a key component of the Model for Improvement widely used within the NHS.

SPC is widely used in the NHS to understand whether change results in improvement and provides an easy way for people to track the impact of improvement projects. 

The SPC visual created using deneb is made up of four rules. If a rule is broken then the data points will show as being of improvement or of concern. This is be determined by the direction of travel; are points going up good or bad?

**Rule 1**. A single point outside the process limit.
Checks to see if point x is outside the process control limit, at either end. This is called an astronomical point. Points are counted in isolation so two points in a row outside the process limit would both count. 
                
**Rule 2**. Two of three data points close to a process limit.
A data point is close to the process limit if it is within two sigma. Data points count if they are either "close" to the process limit AND DO NOT GO ABOVE THE process limit. For the data points to count, two out of three have to meet this criteria. Points which are part of a two out of three sequence but not "close" or over are not shown as improvement or concern. 
                
**Rule 3**. Shift of points above / below the mean line.
Look for x number of points which fall above or below the mean in an unbroken order. X is defined in the specification.json file under the params section. A default of 7 is applied. 
                
**Rule 4**. Run of data points in ascending or descending order.
Look for x number of points ascending or descending in an unbroken order. X is defined in the specification.json file under the params section. A default of 7 is applied. 

SPC charts that match the NHS requirements are not available within Power BI out of the box. As of this writing some solutions for custom visuals are available but usually come with associated costs to use their full functionality. Sometimes SPC calculations are derived within the dataset that is made available for Power BI. This can be a good solution but means custom filtering will not re-calculate the rules and restricts the flexibility of a solution to only pre-calculated combinations. It is also possible to code up the rules in DAX. However this creates a huge number of measures which need to be generated into each model. If a change to the ruleset is needed then it is very complex to maintain. Deneb provides a nice option as the full code can be pasted into the specification and config sections easily and a minimal number of DAX measures need to be passed to the visual. 
                
## Installing Deneb

Deneb can be added to your Power BI file from AppSource. If you are using the Power BI Service and have organisational policies setup you may need Deneb to be added to your "Organizational visuals" menu within the Admin portal. Your Power BI administrator will be able to assist with this if you do not have access to do it yourself. 

![SPC Chart Example](docs/images/PBI%20Service%20Organizational%20visuals.png)

## Setting up Power BI to use Deneb Visual
The deneb visual has been designed to do most of the hard work within the visual itself. This means there is limited amount of setup you have to do with the data and associated DAX measures to get it to work. It also makes it easier to update the visual if any changes are made to the codebase. To get started just paste the config.json and specification.json into the Deneb Power BI visual and add in the mandatory measures. 

## Fields for the Deneb Visual
Fields for the Deneb visual must be named correctly on the values section of the visualisation pane. The following fields should be used:

#### Date
Used in our x-axis to plot data points over time. Should be in a date format. Results will be ordered and evaluated against each other in date sequence. 
#### Result
The result for each data point. Should be passed in as a measure. 
#### TargetDirection (optional)
Should be a value of "High" or "Low". High denotes that points going up in value describe an improvement. Low denotes that points going down in value denote concern. Determines if a data point should be of improvement or of concern. If no target direction is provided then a default will be used defined in the parameter DefaultTargetDirection. 
#### Target (optional)
This is used to draw a target line onto the chart. 

## Specification.json Breakdown
The specification.json makes up the bulk of the visual. The following section describes how parts of the specificaiton.json file implement SPC.

### Params
A number of parameters are defined to make it easier to work with the code.
#### SplitMarkers
Defines if the marker showing when a point is improving, of concern or common cause can be split. Sometimes a marker can be part of both an improvement and concern group. if this is set to true then a marker can show a split circle. If set to false and a data point is part of both an improvement and concern group, it will show as improving. 

![SPC Chart Example](docs/images/SPC%20Chart%20Example%202.png)

#### ConcernColour
Defines the colour of the marker if a concern rule is breached and no improvement rule is breached for the same point. The colour is a Hex representation of the default NHS marker colour. 
#### ImprovementColour
Defines the colour of the marker if a improvement rule is broken. The colour is a Hex representation of the default NHS marker colour. 
#### CommonCause 
Defines the colour of a marker if no concern or improvement rule is breached. The colour is a Hex representation of the default NHS marker colour. 
#### MeanGroupSizeThreshold
The minimum amount of sequential points needed to trigger the mean run rule. Sequential points have to be above or below the threshold to qualify. 
#### TrendGroupSizeThreshold
The minimum amount of sequential points needed to trigger the trend run rule. Sequential points have to be on / above or on / below the threshold to qualify. 
#### DefaultTargetDirection
The default direction for improvement if no direction is provided by the user. This will default to High. 
#### ChartTitle
The default name for the chart title.
#### PinYAxisToZero
Defines if you set to y-axis bottom point to 0 or not. Can be set to True or False.
#### Image Parameters
These are a set of parameters which hold the base64 encoding for the images used for variation and assurance. 

### Transform
To simplify the amount of setup needed the majority of calculations are done in the transform section of the specification.json. This significantly reduces the number of fields that are required for the visual and the amount of DAX to code. 
#### RowReverseOrder
Calculates the reverse order of the data points. Used to help identify the last data point in the series.
#### ImprovementDirection
Calculates the direction of improvement. This is based on the target direction provided as a measure. If no target direction is provided the default is taken from the parameter.
#### Mean
Calculates the mean. The mean is calculated across all points visible in the visual.
#### Prior1Row
The result value for the previous row in date order. Null is used when there is no prior result. 
#### Following1Row
The result value for the next row in date order. Null is used when there is no following result. 
#### MovingAverage
The moving average of each result. This is calculated from the absolute different between the current result and previous result in the ordered dataset.
#### MovingRangeAverage
The mean value of the moving range average.
#### MeanGroupStart
Calculates where each mean group starts in the dataset. This is done by comparing the mean against the current result and then checking the same logic against the prior row. Any change in result between the two triggers a new mean group. 
#### MeanGroup
Sums up the MeanGroupStart derived field to place each result into their own unique group. 
#### MeanScore
Counts the number of results in each mean group. This is used to determine if a result has breached the mean run rule. 
#### LCL
Calculation of the lower confidence interval process limit. This is the mean - ( 2.66 * the moving range average).
#### UCL
Calculation of the upper confidence interval process limit. This is the mean + (2.66 * the moving range average). 
#### OutsideUCL
If a datapoint falls outside the UCL. This is used to trigger the outside confidence limits rule.
#### OutsideLCL
If a datapoints falls outside the LCL. This is used to trigger the outside confidence limits rule.
#### NearUCL
If a datapoint is close to the UCL. 
#### NearUCLScore
Counts the number of datapoints within two of each side of the current result in sequential order what are near the upper confidence limits. This is used to determine if a result has breached the two of three points close to a process limit. 
#### UCLTwoOfThreeBeyondTwoSigma
Determines if a datapoint breaches the rule. For the datapoint to display as improvement or common cause it must breach the rule itself and be part of two datapoints that breach out of three sequential data points.
#### NearLCL
If a datapoint is close to the LCL.
#### NearLCLScore
Counts the number of datapoints within two of each side of the current result in sequential order what are near the lower confidence limits. This is used to determine if a result has breached the two of three points close to a process limit.
#### LCLTwoOfThreeBeyondTwoSigma
Determines if a datapoint breaches the rule. For the datapoint to display as improvement or common cause it must breach the rule itself and be part of two datapoints that breach out of three sequential data points.
#### TrendUp
If the data point is greater than the previous point.
#### TrendUpGroupStart
Calculates where each trend group starts in the dataset. This is done by comparing the trend up field against the current result and then checking the same logic against the prior row. Any change in result between the two triggers a new trend group. 
#### TrendUpGroup
Sums up the TrendUpGroupStart derived field to place each result into their own unique groups. 
#### TrendUpScore
Counts the number of results in each trend group. This is used to determine if a result has breached the mean run rule.
#### TrendUpThreshold
Determines if a datapoint has breached the trend threshold. The trend score is compared to the parameter TrendGroupSizeThreshold to see if breached the threshold. 
#### TrendDown
If the data point is lower than the previous point.
#### TrendDownGroupStart
Calculates where each trend group starts in the dataset. This is done by comparing the trend down field against the current result and then checking the same logic against the prior row. Any change in result between the two triggers a new trend group. 
#### TrendDownGroup
Sums up the TrendDownGroupStart derived field to place each result into their own unique groups. 
#### TrendDownScore
Counts the number of results in each trend group. This is used to determine if a result has breached the mean run rule.
#### TrendDownThreshold
Determines if a datapoint has breached the trend threshold. The trend score is compared to the parameter TrendGroupSizeThreshold to see if breached the threshold. 
#### TrendEqual
Checks to see if the current row is equal to the prior row.
#### IsImprovement
Determines if a data point corresponds to an improvement rule. 
#### IsConcern
Determines if a data point corresponds to a concern rule.
#### IsCommonCause
Determine if a data point corresponds to common cause.
#### PrimarySPCCategory
The primary SPC category for a data point. If a data point is flagged as both improvement and concern it will show as improvement. 
#### IsSecondarySPCCategory
Shows the secondary SPC category. If the parameter SplitMarkers is set to true then when a data point is flagged as both improvement and concern it will show as concern (PrimarySPCCategory will be shown as improvement). If the parameter SplitMarkers is set to false then this will match the results of PrimarySPCCategory. 
#### ImprovementComments
Generates the improvements comments for a data point. 
#### ConcernComments
Generates the concern comments for a data point. 
#### SPCCategoryToolTip
Generates the text which will be shown on the tooltip for the SPC category. This will combine the text generate in the fields ImprovementComments and ConcernComments. 
#### SPCImageUrl
Calculates the base64 image encoding to use for variation. This will only be generated for the last datapoint in the series.
#### SPCImageTooltip
Calculates the tooltip text for the variation tooltip. This will only be generated for the last datapoint in the series.
#### Capability
Calculates the capability of the data series. 
#### CapabilityImageUrl
Calculates the base64 image encodoing to use for assurance. This will only be generated for the last datapoint in the series. 
#### CapabilityImageTooltip
Calculates the tooltip text for the assurance tooltip. This will only be generated for the last datapoint in the series. 

### Layer
The visual is made up of a number of different layers that contribute to the end design. There are lines drawn for the UCL, LCL, mean and if applicable the target. The results are also shown as a line. For each result datapoint a marker is shown which represents it's status against the SPC rules. Datapoints which breach rules will be shown as either improvement or concern. If a datapoint does not breach any rule then it will be shown as having common cause. 

## Limitations
The Deneb visual can take a second or so to initially load on the screen. This appears to only effect the initial rendering; if you update the context filter the visual operates in it updates in good time. Be careful when placing multiple of these visuals on the same page as they appear to render initially in sequence. The more visuals there are, the longer it will take to initially render them all to the screen. The visual should be used when you cannot pre-calculate the rules at the source, such as SQL. 
