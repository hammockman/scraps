Joe90 Measurement Uncertainty
=============================


Background - Joe90
----------

The Joe90 system has been in use at Scion since 2007, with a number of refinements being implemented on top of the original system designed by Joescan for Weyerhaeuser:

- replacement of the original point cloud post-processing algorithm,
- modification of the board mounting system,
- inclusion of weight and resonance measurement equipment,
- addition of a motor drive,
- replacement of the original user interface software and data storage mechanism.

To date, no comprehensive experiment has been conducted to determine the quality of measurements coming from the system, although a variety of ad-hoc tests have been conducted, including:

- the repeated scanning of a number of individual boards, 
- scanning of reference boards (white beam and DG3000) at the start of each batch of scans (typically at the start of each day).

Currently the joe90 system is being used to assess the approach to equilibrium of a set of boards for a commercial client (SWI, LS.1.5). In order to do this efficiently, good quality estimates of the uncertainty associated with measures of warp (particularly crook and bow) are required.

As well as , other reasons for performing a repeatability and reproducibility experiment include:

- incorporation in the Joe90 publication currently in preparation
- serving as a basis for QC/QA in the future
- the dataset collected can be used to guide refinement/tuning of the post-processing algorithm
- serve as test set for a fully numerical approach to algorithm uncertainty (3D board model - simulated noisy point cloud - recovered board shape)

Background - Repeatability and Reproducibility
----------------------------------------------

Measurement systems analysis is a large and complex topic (see references). In the context of measurement systems analysis, repeatability and reproducibility are defined as:

Repeatability
: is the measurement variability exhibited when the same item is measured repeatedly with the same equipment by the same operator and within a period of time short enough to exclude the unwanted influence of factors that cannot be guaranteed constant, e.g. environment. Recalibration, unless an essential part of every single measurement, should not occur during repeatability testing.

Reproducibility 
: is the additional measurement variability found when the same item is measured repeatedly under conditions other than those used for repeatability testing, for example over lengthy periods of time, after recalibration, with different operators, in different laboratories or with different equipment. 

Here we are interested in both repeatability, and a limited version of reproducibility.

A wide range of factors potentially contribute to Joe90 measurement uncertainty:

- board length, size, shape including
    - presence and degree of wane
    - dimensional variation induce by sawing
    - cross-sectional departures from rectangular (wane, cup, sawing)
- positioning of the board relative to the supports, including
    - orientation of the board relative to gravity
    - the distance from acoustic hammer to board end
- motion of the board relative to supports during scan
- changes in straightness of the transport rail (if not corrected for)
- changes in position of supports (relative to one another and relative to start position)
- board surface finish and surface moisture content (affects where laser light reflects from)
- Joescan camera calibration (accounts for relative positions and orientations of the cameras)
- load cell calibration
- post-processing algorithm
    - the algorithm itself
    - tuning parameters
- the summary measure being considered (e.g max(bow) sensitive to small changes in S shaped boards)
- temperature and humidity changes (over short periods of time)

Quantifying the contribution of these factors to overall measurement uncertainty is compounded by the changes in the boards being measured due to changes in temperature, moisture content and inelastic deformation (e.g. creep under self-weight).


Proposed Approach
-----------------

There are essentially two approaches to measurement system analysis. 

The first takes a bottom-up perspective, involves a so-called uncertainty budget and derives overall uncertainty from a mathematical model of the measurement process and nominal uncertainties of constituent data (e.g. the stated uncertainty of point positions coming from the JS-20 scan heads is &pm;0.75 mm). The complexity of the Joe90 measurement equipment and procedure effectively rules this approach out.

The second approach, and the approach adopted here, makes use of a statistical model to interpret results from a designed experiment.

Trueness of the Joe90 system has been proved previously[^ffr-ground-truth]. Here the focus is on measurement uncertainty, particularly the uncertainty of the various warp measures. 

[^ffr-ground-truth]: As part of the FFR PQPV study, traditional methods were used to independently assess warp, density and stiffness of a set of boards for comparison with joe90. In all cases the results were found to be in agreement.


### Board Selection
Select a set of boards to represent typical material tested. These boards should exhibit

- a range of bow crook and twist levels (two levels of each, not necessarily in different boards)
- a range of waney-nesses

No attempt will be made to control other board characteristics (e.g. surface finish, knottiness).

Initially, a set of 6 boards will tested, presuming boards with suitable combinations of characteristics can found. Should analysis indicate that measurement uncertainty depends strongly on board characteristics (e.g. warp level) further boards may need to be tested.

To keep the number of scans to a minimum, only a single board size (100x40's) and length will be considered .

Selected boards will all be reduced to the maximum length which can be fitted into an environmentally controlled room. (2.4m?)[^board-length]. 

[^board-length]: While the boards whose measurement uncertainty is most urgently required are 5m long, these boards cannot be kept in a controlled environment. It is possible that board length affects measurement uncertainty

After selection, and for a minimum of 1 week prior to commencing scanning, the selected boards will be placed into a controlled environment. Temperature and RH within the conditioning environment will be recorded continuously for the duration of the experiment.


### Board Scanning

Scanning will take place in batches. Each batch will be scanned by one of two operators. Initially three batches per operator are planned[^batch-timing]. Repeatability conditions will be expected to hold within batches (i.e. same operator, minimal elapsed time, no recalibration).

[^batch-timing]: Each batch should be able to be executed within a half-day. Ideally all batches would be run in a single week.

Before commencing a scan batch:

- use the JSdiag app to calibrate the cameras,
- calibrate the load cells using the normal procedure,
- scan the white and dg3000 boards prior to scanning anything else.
- measure the moisture content of each board using an electrical resistance probe inserted at a minimum of 3 locations distributed uniformly along the board length. Do not reuse previous probe positions [Check this with SteveR].

Within each batch each board will be scanned 6 times in each of 2 orientations, with 2 pairs of rescans for each of the 3 re-placements on the supports. The re-placements should not be sequential (i.e. at least one other board should be scanned between repeats).

The two board orientations to be considered are both 'joist-like' (i.e. wide face vertical) but rotated 180&deg; about the board axis[^orientations].

[^orientations]: Other board orientations (plank-like, flipped-end-to-end), while possible, are deemed unlikely to occur by accident and hence, in the interest of economy, are not considered herein.

When outside of the controlled environment, and when not being scanned, boards will be kept in polythene socks to prevent moisture movement.

Time between batches should be minimized, and as much as possible, batches should be performed in similar environments by, for example, starting and stopping at the same time of day, avoiding extreme weather, etc. Ambient temperature and RH should be recorded for each batch.

Between batches no adjustment to equipment will be made other than specified above. This includes, for example, the distance between support positions. If such changes should occur inadvertently, the entire sequence of scans will be re-run from the start.



### Post-Processing

The scan data will be post-processed using Joescan3 algorithm with the current set of tuning parameters. 

Initially, board orientation will be corrected for by rotating the point cloud.

### Statistical Analysis

After removing any outliers, a mixed effect model will be fitted to the data. The lme4 package in R will be used. Initially the model to be fitted will be (in the notation used by `lmer()` )
$$ y \sim 1 + (1|board) + (1|replacement) + (batch)$$ 
where the measurand $y$ is one of:

- mass
- resonance
- volume
- dimensions
- mid-point or RMS bow
- mid-point or RMS crook
- total or RMS twist

Further models will be considered based on inferences drawn, e.g. operator will be included should residuals be found to vary significantly between operators.

Questions to be addressed by the analysis include:

- estimates for the uncertainty of each of the measurands.
- do the measurement uncertainties depend on the value of the measurand?
- to what extent do the measurement uncertainties depend on the values of other factors (e.g. presence of wane)?
- the need for, or benefits of, collecting further data by either extending the selected set of boards or performing further scan batches.
- can differences in board orientation be accounted for by simply rotating the resulting point cloud?

### Reporting

A brief report will be prepared summarizing the measurements made and the measurement uncertainties determined. The report will not describe the data processing or analysis methods used in any detail.


Costs
-----

| activity  | time (hrs) | personnel | cost |
|:-------------------|-----:|:---|---:|
| Sample selection |    4 | JL  |  |
| Scanning (432 scans @ 100 s/scan[^scantime] + setup)  |  18 | JL, RP
| Post-processing of scan data |  4 | JH
| Statistical analysis  | 24 | JH, MK? |  |
| Report  |  8 | JH, JL, GE |
| **Total**  |  |  | ~$10k[^cost]

[^cost]: $10k seems like a reasonable amount to spend given the cost of collecting the LS.1.5 dataset. Scan numbers have been scaled accordingly.



[^scantime]: An analysis of the last 5000 scans indicate a median time bewteen scans of 96s. The majority of these successive scans are of different boards, thus representing a conservative estimate for the time between successive scans of the same board. 

References
----------
* [JCGM 100:2008 Evaluation of measurement data — Guide to the expression of uncertainty in measurement](http://www.iso.org/sites/JCGM/GUM-introduction.htm)
* [Guidelines for Evaluating and Expressing the Uncertainty of NIST Measurement Results](http://www.nist.gov/pml/pubs/tn1297/index.cfm)
* http://en.wikipedia.org/wiki/Measurement_systems_analysis
* [ISO 5725 Accuracy of Measurement Methods and Results](https://www.iso.org/obp/ui/#iso:std:iso:5725:-1:ed-1:v1:en)
* ASTM E2782 Standard Guide for measurement systems analysis

