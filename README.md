# Uber Ride Bookings Analysis -- Power BI Dashboard

## 1. Project overview

This project analyses ride-booking activity using a Power BI dashboard
built from the **NCR Ride Bookings** dataset. The report is designed to
help users understand booking volume, booking outcomes, vehicle-type
performance, revenue, cancellations and customer/driver ratings.

The workflow covered in this project is:

1.  Import the source CSV into Power BI.
2.  Inspect and clean the data in **Power Query Editor**.
3.  Apply the cleaned data to the Power BI model.
4.  Create measures and configure data modelling.
5.  Build report pages, visuals, slicers and navigation.
6.  Use the dashboard to explore operational and business performance.

**Project files** - `ncr_ride_bookings.csv` -- source dataset. -
`Uber.pbix` -- Power BI report file. - `README.md` -- project
documentation.

## 2. Dataset description

The source file contains **150,000 rows and 21 columns**. Each row
represents a ride-booking record. The data covers **1 January 2024 to 30
December 2024**.

### Main fields

  -----------------------------------------------------------------------
  Field                               Meaning
  ----------------------------------- -----------------------------------
  Date, Time                          Date and time when the booking was
                                      recorded

  Booking ID                          Unique booking reference (check
                                      duplicates during quality
                                      validation)

  Booking Status                      Outcome of the booking, such as
                                      Completed, Incomplete or Cancelled

  Customer ID                         Customer reference

  Vehicle Type                        Vehicle category selected/requested

  Pickup Location, Drop Location      Starting and destination areas

  Avg VTAT                            Average vehicle time to arrive, as
                                      represented in the source

  Avg CTAT                            Average customer trip time, as
                                      represented in the source

  Cancelled Rides by Customer         Customer-cancellation
                                      indicator/count field

  Reason for cancelling by Customer   Reason recorded for customer
                                      cancellation

  Cancelled Rides by Driver           Driver-cancellation indicator/count
                                      field

  Driver Cancellation Reason          Reason recorded for driver
                                      cancellation

  Incomplete Rides                    Incomplete-ride indicator/count
                                      field

  Incomplete Rides Reason             Reason recorded for an incomplete
                                      ride

  Booking Value                       Booking amount/value

  Ride Distance                       Distance recorded for the ride

  Driver Ratings                      Rating given to the driver

  Customer Rating                     Customer rating recorded for the
                                      ride

  Payment Method                      Payment method used
  -----------------------------------------------------------------------

### Source-data profile

  Item                                         Value
  ----------------------- --------------------------
  Total rows                                 150,000
  Total columns                                   21
  Date range                2024-01-01 to 2024-12-30
  Completed bookings                          93,000
  Cancelled by driver                         27,000
  Cancelled by customer                       10,500
  No driver found                             10,500
  Incomplete bookings                          9,000

The booking-status counts above are from the supplied CSV. They are
useful for validating the report, but dashboard values can change when
slicers or filters are applied.

## 3. Data preparation in Power Query Editor

Power Query is the ETL layer of Power BI: **Extract, Transform and
Load**. It allows data to be inspected and prepared before it is loaded
into the data model. Keeping preparation steps in Power Query makes the
process repeatable whenever the source file is refreshed.

### 3.1 Connect to the CSV

1.  Open Power BI Desktop.
2.  Select **Home → Get data → Text/CSV**.
3.  Select `ncr_ride_bookings.csv`.
4.  Preview the file and choose **Transform Data** rather than loading
    it immediately.
5.  Power Query Editor opens with the source table.

### 3.2 Inspect the data

Before changing values, inspect: - Column names and whether they are
meaningful. - Data types (date, time, text, decimal and whole number). -
Blank/null values. - Duplicate booking references. - Unexpected spaces
or inconsistent text. - Whether numeric columns contain valid numeric
values. - Whether cancellation and incomplete-ride fields are populated
only for the relevant booking outcomes.

This step matters because incorrect types and inconsistent values can
cause incorrect aggregations, sorting, relationships and measures.

### 3.3 Data types

Set types according to the meaning of each column:

  -----------------------------------------------------------------------
  Column(s)                           Recommended type
  ----------------------------------- -----------------------------------
  Date                                Date

  Time                                Time

  Booking ID, Customer ID             Text

  Booking Status, Vehicle Type,       Text
  locations, reasons, Payment Method  

  Avg VTAT, Avg CTAT                  Decimal number

  Booking Value, Ride Distance,       Decimal number
  Driver Ratings, Customer Rating     

  Cancellation and incomplete-ride    Whole number, where appropriate
  indicator fields                    
  -----------------------------------------------------------------------

**Why this matters:** A date stored as text may not sort
chronologically. An ID stored as a number may lose leading zeros or be
aggregated incorrectly. A rating or amount stored as text cannot be
reliably averaged or summed.

### 3.4 Text standardisation

For text columns, a sound cleaning workflow is: - Apply **Transform →
Format → Trim** to remove leading and trailing spaces. - Apply **Clean**
where non-printing characters are present. - Standardise inconsistent
category spelling/capitalisation if any exists. - Check Booking ID and
Customer ID values for unwanted quote characters or spaces. Remove such
characters only if they are formatting artefacts and the identifiers are
meant to be stored without them.

Do not change valid category names or identifiers merely to make them
look different. Preserve the original business meaning.

### 3.5 Missing values

The source has many blank values. These are not automatically errors.
Several columns are only relevant for particular booking outcomes.

Examples from the source: - `Booking Value`, `Ride Distance` and
`Payment Method` each have 48,000 blanks. - `Driver Ratings` and
`Customer Rating` each have 57,000 blanks. - Cancellation-reason fields
are populated only for the relevant cancellation type. -
`Incomplete Rides Reason` is populated for incomplete bookings.

A blank rating for a cancelled ride is usually **not applicable**,
rather than a rating of zero. Replacing it with zero would lower the
average incorrectly. Similarly, do not replace missing booking values or
distances with zero unless the business definition explicitly says that
zero is correct.

Recommended handling: 1. Check null counts using **View → Column quality
/ Column distribution / Column profile**. 2. Compare nulls with
`Booking Status`. 3. Retain meaningful nulls for fields that do not
apply to a particular booking outcome. 4. Remove rows only when they are
genuinely unusable and the reason is documented. 5. If a replacement is
required, choose a business-appropriate value and document the rule.

### 3.6 Duplicate and validity checks

-   Check duplicate `Booking ID` values. The supplied source contains
    150,000 rows and 148,767 distinct Booking ID values, so duplicates
    should be investigated before deciding whether to remove them.
-   Check that dates fall within the expected period.
-   Check that ratings are within the valid scale (the supplied data
    ranges from 3 to 5).
-   Check that ride distance and booking value are numeric and
    non-negative.
-   Check that each booking has a sensible status and that
    cancellation/incomplete reason fields are consistent with that
    status.

**Important:** Do not automatically remove duplicate IDs without
checking whether they are duplicate records or legitimate repeated
source records. Decide the grain of the analysis first.

### 3.7 Apply the transformations

After reviewing and applying the required transformations, select **Home
→ Close & Apply**. Power BI loads the prepared table into the semantic
model.

> **Reproducibility note:** The supplied PBIX report file exposes the
> report pages and visual queries, but it does not provide a readable
> Power Query Applied Steps/M script in the inspected package.
> Therefore, this README explains the appropriate cleaning workflow for
> this exact dataset; it does not claim that every listed optional check
> or transformation was definitely applied in the original PBIX. To make
> the project fully auditable, add the exact Applied Steps names or
> paste the M script into this section.

## 4. Data modelling

Data modelling defines how data is organised and how calculations behave
in the report.

### 4.1 Main table

The report visuals query a table named **`ncr_ride_bookings`**. It
contains the booking-level fields used throughout the report.

The PBIX structure shows report queries against this table and
date-hierarchy fields. The report file does not expose enough readable
model metadata here to confirm every relationship or every hidden model
setting. If you later add a separate calendar table, create a
one-to-many relationship from the calendar's unique Date column to
`ncr_ride_bookings[Date]`, and mark the calendar table as a date table.

### 4.2 Why a date hierarchy is useful

A date hierarchy lets a user drill from: - Year → Quarter → Month → Day.

This makes it possible to move from a high-level annual trend to a more
detailed monthly or daily view. Use a proper calendar table for more
advanced time-intelligence calculations and consistent date filtering.

### 4.3 Measures versus calculated columns

A **calculated column** is evaluated row by row and stored in the model.
It is useful for a row-level category or flag.

A **measure** is evaluated when a visual is queried. Its result changes
according to the current filter context, such as selected date, vehicle
type or payment method. Measures are generally preferable for KPIs and
aggregated business calculations.

The report contains a measure named **`CanceledPercentage`**, used on
the Cancellation page. Its exact DAX expression could not be recovered
from the readable report layout, so the formula below is an illustrative
pattern only---not a claim about the exact formula in the PBIX:

``` dax
CanceledPercentage =
DIVIDE ( [Cancelled Bookings], [Total Bookings], 0 )
```

For a reliable percentage, define the numerator and denominator
consistently. For example, decide whether "cancelled" includes customer
cancellations only, driver cancellations only, or both. Also decide
whether "No Driver Found" is a cancellation category for the business
question. Document that definition next to the measure.

### 4.4 Aggregations used in the report

The report uses aggregations such as: - **Count / distinct count of
Booking ID** for booking volume. - **Sum of Booking Value** for
revenue/value analysis. - **Sum of Ride Distance** for distance
analysis. - **Average of Customer Rating** and **Average of Driver
Ratings** for rating KPIs. - Cancellation-related counts and the
`CanceledPercentage` measure.

Power BI can create implicit aggregations by dragging a numeric field
into a visual. Explicit DAX measures are usually easier to name, reuse
and maintain.

## 5. Dashboard guide -- page-by-page explanation

The PBIX contains six report pages: **Home Page, Overall, Vechical Type,
Renvenue, Cancellation and Ratings**. The spellings shown here match the
page labels in the supplied report.

### 5.1 Home Page

This is the landing/navigation page. It provides the entry point to the
report. The report also contains action-button visuals on the analysis
pages, which appear to support navigation between sections.

**How to use it:** Select the relevant navigation button to move to the
page that answers your question. A clear landing page helps a viewer
understand where to begin rather than being confronted with every chart
at once.

### 5.2 Overall

**Purpose:** Give a quick summary of booking activity and booking
outcomes.

The page includes: - A **line chart** showing booking count over the
date hierarchy. - A **pie chart** showing the distribution of Booking
Status. - A **Date slicer** for selecting the analysis period. - A
**booking-count card**.

**How to interpret the visuals:** - The line chart shows how booking
volume changes over time. Peaks may indicate high-demand periods; drops
may indicate lower demand or operational disruption. A trend is more
meaningful when compared across equivalent periods. - The status pie
chart shows the share of each outcome. A large Completed share indicates
that many bookings reach completion; cancellation, incomplete and
no-driver outcomes highlight potential service issues. - The date slicer
narrows the analysis to a selected period. Other visuals on the page
respond to the selection when interactions are enabled. - The booking
card gives a headline count for the currently selected filter context.

**Questions this page can answer:** - How many bookings are recorded in
the selected period? - Is booking volume increasing or decreasing? -
What proportion of bookings are completed versus unsuccessful? - Does
the booking-status mix change over time?

### 5.3 Vechical Type

**Purpose:** Compare performance across vehicle categories.

The page contains multiple KPI cards based on **Booking Value** and
**Ride Distance**, with visual-level filters on Vehicle Type, plus a
Date slicer. The repeated cards allow vehicle categories to be compared
side by side.

Vehicle categories in the source include Auto, Go Mini, Go Sedan, Bike,
Premier Sedan, eBike and Uber XL.

**How to interpret the page:** - Booking Value cards show the total
booking value for the vehicle category represented by each card. - Ride
Distance cards show distance aggregation for the relevant vehicle
category. Check the displayed aggregation label: a sum answers a
different question from an average. - Compare categories within the same
date selection. Otherwise, differences may be caused by different time
periods rather than vehicle performance. - A high total booking value
can be caused by more trips, higher average fares, or both. It does not
automatically mean that the vehicle category has the highest
profitability.

**Questions this page can answer:** - Which vehicle category contributes
the most booking value? - Which category accounts for the most ride
distance? - How does the comparison change when the date range changes?

### 5.4 Renvenue

**Purpose:** Explore booking value by payment method and over time, and
inspect customer-level booking value.

The page includes: - A **column chart** of Booking Value by Payment
Method. - A **Date slicer**. - A second **column chart** using the date
hierarchy and Ride Distance. - A **table** containing Customer ID and
Booking Value.

**How to interpret the visuals:** - The payment-method chart compares
the total booking value associated with each payment type. It helps
identify the payment channels that contribute most to recorded booking
value. - The date-based chart provides a time view of ride distance. It
should be read as a distance trend, not as a revenue trend, because the
plotted field is Ride Distance. - The customer table allows
customer-level booking value to be inspected. Sort the Booking Value
column to find higher-value customers within the current filter
context. - The Date slicer helps compare payment and distance patterns
across periods.

**Questions this page can answer:** - Which payment method accounts for
the highest booking value? - How does ride distance change across the
selected date hierarchy? - Which customers have the largest recorded
booking value?

**Metric caution:** Booking Value is a gross booking-value field in this
dataset. It should not be described as profit unless costs, commissions,
refunds and other relevant expenses are included.

### 5.5 Cancellation

**Purpose:** Understand why bookings are cancelled and monitor
cancellation performance.

The page includes: - A pie chart for **Reason for cancelling by
Customer**, using the customer-cancellation field. - A pie chart for
**Driver Cancellation Reason**, using driver cancellation information. -
Date filtering. - Booking-related KPI cards. - The
**`CanceledPercentage`** measure.

**How to interpret the visuals:** - The customer-reason chart breaks
down the reasons customers give for cancelling. The largest segment
identifies the most frequently recorded reason in the selected period. -
The driver-reason chart shows the distribution of driver cancellation
reasons. It can reveal recurring operational or customer-related
issues. - KPI cards provide headline booking counts under their
configured filters. - `CanceledPercentage` expresses cancellations as a
percentage according to the DAX definition used in the model. Always
check its numerator and denominator before interpreting it.

**Questions this page can answer:** - What are the most common customer
cancellation reasons? - What are the most common driver cancellation
reasons? - Does the cancellation percentage change over time? - Are
cancellation patterns concentrated in a particular period?

**Metric caution:** Customer cancellations and driver cancellations are
separate fields. Avoid adding them blindly if a booking could be
represented in more than one category. Make the cancellation definition
explicit and use a consistent booking-level denominator.

### 5.6 Ratings

**Purpose:** Review customer and driver rating performance.

The page contains KPI cards using the **Average of Customer Rating** and
**Average of Driver Ratings**, with Vehicle Type filters on the cards.

**How to interpret the visuals:** - Customer Rating reflects the rating
recorded for the customer/ride experience in the dataset. - Driver
Ratings reflects the driver-rating field. - Average ratings make
categories easier to compare, but the number of rated rides also
matters. A category with a high average based on a small number of
ratings may be less reliable than one with many ratings. - Blank ratings
should generally remain blank rather than being converted to zero,
because zero would distort the average.

**Questions this page can answer:** - Which vehicle categories have
higher average customer ratings? - Which vehicle categories have higher
average driver ratings? - Is there a noticeable difference between
customer and driver rating averages?

## 6. Filters, slicers and interactions

A slicer is a visual filter that lets a report user select a subset of
the data. The report includes Date slicers on the analysis pages.
Selecting a date range should update connected visuals according to the
report's interaction settings.

When exploring the report: 1. Start with the full date range. 2. Select
a shorter date range. 3. Compare the KPIs and charts before and after
filtering. 4. Clear the slicer to return to the full view. 5. Use visual
selections (such as a pie-chart segment) to cross-filter other visuals
if interactions are enabled.

If a visual does not respond as expected, check **Format → Edit
interactions** in Power BI Desktop.

## 7. Business insights supported by this dashboard

The dashboard is designed to support several operational questions:

-   **Demand monitoring:** booking trends help identify busy and quiet
    periods.
-   **Service quality:** booking status distribution highlights
    completion and unsuccessful booking outcomes.
-   **Fleet/vehicle mix:** vehicle-type comparisons show how booking
    value and distance are distributed across categories.
-   **Payment behaviour:** payment-method analysis shows the
    contribution of each recorded payment channel to booking value.
-   **Cancellation management:** cancellation reasons can help
    prioritise operational improvements.
-   **Experience monitoring:** customer and driver ratings provide a
    view of recorded service feedback.

These are analytical uses, not causal conclusions. For example, a high
cancellation count does not by itself explain why cancellations
occurred; the reason breakdown and additional operational data are
needed.

## 8. Validation and quality checks

Before publishing or presenting the dashboard, validate the following:

-   Total row count is consistent with the intended source extract.
-   Booking IDs are checked for duplicates and the chosen row grain is
    documented.
-   Date and time columns have correct data types.
-   Booking Value and Ride Distance use the intended aggregation.
-   Rating cards use **Average**, not Sum.
-   Blank ratings are not treated as zero.
-   Cancellation percentages use a clearly defined numerator and
    denominator.
-   Date slicers filter all intended visuals.
-   Vehicle-type cards each have the correct vehicle filter.
-   The customer table is sorted and formatted appropriately.
-   Titles, units, currency formatting and decimal places are clear.
-   Navigation buttons work in Reading view.

## 9. How to open and use the project

1.  Install/open **Microsoft Power BI Desktop**.
2.  Open `Uber.pbix`.
3.  If Power BI requests a source-file path, browse to
    `ncr_ride_bookings.csv` and refresh.
4.  Wait for the model and visuals to load.
5.  Use the page tabs or navigation buttons to move between report
    sections.
6.  Use the Date slicers and visual selections to explore the data.
7.  Save a copy before changing measures, relationships or report
    design.

## 10. Limitations and future improvements

Possible improvements for a future version:

-   Add a dedicated Calendar table and use it consistently for all date
    filtering.
-   Replace repeated implicit aggregations with clearly named explicit
    DAX measures.
-   Add total bookings, completed bookings, customer cancellations,
    driver cancellations, incomplete bookings and no-driver-found
    measures with precise definitions.
-   Add completion rate and cancellation rate KPIs with documented
    business logic.
-   Add average booking value, average ride distance and average
    ratings.
-   Add a breakdown of incomplete-ride reasons.
-   Add pickup and drop-off location analysis (for example, top pickup
    areas and popular routes).
-   Add hourly/day-of-week demand analysis using the Time field.
-   Add tooltips, drill-through pages and accessible visual titles.
-   Document the exact Power Query Applied Steps and the full DAX
    definitions in this README.
-   Consider a star schema if the project grows: a booking fact table
    with separate Date, Customer, Vehicle, Location and Payment
    dimensions where appropriate.

## 11. Conclusion

This Power BI project turns raw ride-booking records into an interactive
report for monitoring booking activity, vehicle-type performance,
booking value, ride distance, cancellation reasons and ratings. Power
Query prepares the source data, the semantic model provides a consistent
structure for analysis, DAX measures define reusable business
calculations, and report visuals make the results easier to explore.

The most important practice is to keep metric definitions clear. A
dashboard is trustworthy when its data preparation, aggregation rules,
filters and calculations are understood---not just when its charts look
attractive.
