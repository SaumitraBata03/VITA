# VITA

### AI-Powered Contactless Emergency Health Screening & Monitoring

**SENSE → ANALYSE → CORRELATE → FLAG**

A modular, contactless screening prototype concept combining **mmWave
radar**, **thermal sensing**, signal processing, and confidence-aware
sensor fusion to support rapid preliminary assessment.

> **VITA is a research and engineering prototype concept---not a medical
> device or diagnostic system.** It does not diagnose disease, determine
> whether a person is infectious, or replace qualified healthcare
> professionals.
:::

------------------------------------------------------------------------

## Table of contents

-   [The idea](#the-idea)
-   [Why VITA?](#why-vita)
-   [How it works](#how-it-works)
-   [System architecture](#system-architecture)
-   [Sensing technologies](#sensing-technologies)
-   [AI and sensor fusion](#ai-and-sensor-fusion)
-   [Hardware concept](#hardware-concept)
-   [User interface concept](#user-interface-concept)
-   [Implementation roadmap](#implementation-roadmap)
-   [Validation plan](#validation-plan)
-   [Performance metrics](#performance-metrics)
-   [Safety, privacy, and limitations](#safety-privacy-and-limitations)
-   [Potential applications](#potential-applications)
-   [Repository structure](#repository-structure)
-   [Getting started](#getting-started)
-   [Team and contributions](#team-and-contributions)
-   [Disclaimer](#disclaimer)

------------------------------------------------------------------------

## The idea

In emergencies and high-throughput care environments, staff may need to
perform quick preliminary checks while managing limited time and
resources. VITA explores whether two complementary contactless sensing
technologies can provide useful physiological observations from a person
standing in a defined sensing zone.

-   **mmWave radar** observes small movements, including chest motion
    associated with breathing, when the person and environment permit
    reliable measurement.
-   **Thermal sensing** observes surface-temperature patterns in a
    selected region of interest.
-   **Signal-quality assessment** checks whether the collected data is
    usable.
-   **Sensor fusion** combines available features while accounting for
    their quality.
-   **The interface** communicates the observation and whether a repeat
    measurement may be needed.

VITA is designed around a controlled, single-person sensing zone for an
initial prototype. This makes sensor alignment, repeatable experiments,
and performance evaluation more manageable before considering wider
deployment.

## Why VITA?

Conventional screening workflows can involve repeated handling, direct
interaction, and separate measurements. A single measurement---such as a
surface-temperature reading---also cannot provide a complete picture of
a person's health.

VITA explores a different workflow: acquire multiple contactless
observations, assess the quality of each, and present a cautious
preliminary result instead of treating any one sensor as definitive.

### Design principles

  -----------------------------------------------------------------------
  Principle                           How VITA applies it
  ----------------------------------- -----------------------------------
  Contactless observation             Uses radar and thermal sensing
                                      without requiring a wearable sensor

  Complementary measurements          Analyses respiratory-motion
                                      features, movement, and thermal
                                      observations separately

  Quality before conclusions          Flags poor positioning, excessive
                                      movement, or unreliable sensor data

  Explainable processing              Starts with measurable features and
                                      interpretable models before
                                      evaluating more complex AI

  Modular hardware                    Separates the sensor head, display,
                                      processing enclosure, column, and
                                      base

  Human oversight                     Presents decision-support
                                      information rather than an
                                      automated diagnosis
  -----------------------------------------------------------------------

## How it works

``` text
        PERSON ENTERS THE SENSING ZONE
                       |
             Presence / Position Check
                       |
          +------------+------------+
          |                         |
     mmWave Radar              Thermal Sensor
          |                         |
   Radar Signal Capture       Thermal Frames
          |                         |
   Filtering & Denoising      ROI Selection
          |                         |
   Respiration / Motion       Temperature Features
          |                         |
          +------------+------------+
                       |
           Signal-Quality Assessment
                       |
             Feature-Level Fusion
                       |
       Confidence / Validity Assessment
                       |
       +---------------+----------------+
       |               |                |
  Observation     Repeat Check     Further Review
  appears usable  recommended      may be appropriate
```

The labels above describe a proposed workflow, not clinically validated
decision categories. Final output rules must be defined and evaluated
during development.

## System architecture

VITA can be organised into five main subsystems:

1.  **Sensing head** --- mmWave radar module and thermal sensor mounted
    with a stable, adjustable orientation.
2.  **Processing electronics** --- embedded computer or single-board
    computer for acquisition, signal processing, and interface control.
3.  **Mechanical chassis** --- enclosure, mounting brackets, vertical
    column, and weighted base.
4.  **Software pipeline** --- sensor drivers, data-quality checks,
    feature extraction, fusion logic, logging, and status handling.
5.  **User interface** --- displays sensor state, measurement quality,
    estimated features, and an appropriately cautious screening status.

``` text
[mmWave Radar] ─┐
                ├─> [Acquisition] -> [Quality Checks] -> [Feature Extraction]
[Thermal Sensor]┘                                             |
                                                              v
[Display / UI] <─ [Status & Logging] <─ [Confidence-Aware Fusion]
```

## Sensing technologies

### 1. mmWave respiration and movement sensing

Breathing causes small periodic changes in chest position. A radar can
detect changes in the reflected signal and, under suitable conditions,
extract a component associated with respiratory motion.

For a monostatic radar, an idealised relationship between radial
displacement and round-trip phase change is:

\[ `\Delta `{=tex}R = `\frac{\lambda \Delta \phi}{4\pi}`{=tex} \]

where:

-   (`\Delta `{=tex}R) is the estimated radial displacement,
-   (`\lambda`{=tex}) is the radar wavelength,
-   (`\Delta `{=tex}`\phi`{=tex}) is the phase change.

For an illustrative carrier frequency of 60 GHz:

\[ `\lambda `{=tex}= `\frac{c}{f}`{=tex}
`\approx `{=tex}`\frac{3 \times 10^8}{60 \times 10^9}`{=tex} =
5`\text{ mm}`{=tex} \]

This illustrates why mmWave radar can be sensitive to small movements.
It does **not** guarantee a particular displacement accuracy; actual
performance depends on the radar hardware, signal processing, range,
target orientation, multipath, and movement.

If the extracted respiratory frequency is (f_r) hertz, the estimated
respiration rate is:

\[ RR = 60 f_r `\quad `{=tex}`\text{breaths/minute}`{=tex} \]

For example, (f_r = 0.25`\text{ Hz}`{=tex}) corresponds to an estimated
rate of 15 breaths/minute. In practice, VITA should estimate respiration
across multiple cycles and report the measurement window and signal
quality. Gross body motion may make the estimate invalid and should
trigger a quality warning or repeat observation.

### 2. Thermal sensing

The thermal module captures surface-temperature-related data without
physical contact. Software selects a region of interest (ROI) and
extracts a robust summary from the available pixels.

A simple ROI mean is:

\[ T\_{`\mathrm{ROI}`{=tex}} =
`\frac{1}{N}`{=tex}`\sum`{=tex}\_{i=1}\^{N} T_i \]

where (T_i) represents the temperature value for pixel (i), and (N) is
the number of valid pixels in the ROI.

Thermal readings can be affected by ambient conditions, sensor
calibration, distance, viewing angle, emissivity, and the selected body
region. Skin-surface temperature is not the same as core body
temperature. Any temperature threshold or interpretation must therefore
be validated for the specific sensor and intended use; a single thermal
value must not be treated as a diagnosis.

### 3. Motion and signal-quality assessment

Movement can contaminate radar respiration estimates and can also affect
the consistency of thermal measurements. VITA should assess signal
quality before combining features.

Potential quality indicators include:

-   Presence and target-position confidence
-   Radar signal-to-noise or periodicity measures
-   Gross-motion level during the observation window
-   Number of usable respiratory cycles
-   Thermal ROI validity and stability
-   Sensor availability, calibration state, and data freshness

When data quality is insufficient, the system should prefer **"Repeat
measurement recommended"** or **"Unable to assess reliably"** over a
confident-looking but unsupported result.

## AI and sensor fusion

VITA's fusion pipeline is intended to combine complementary features
while preserving the distinction between an unusual observation and an
unreliable measurement.

An illustrative weighted score is:

\[ S = w_r R + w_t T + w_m M \]

where (R), (T), and (M) are appropriately normalised radar, thermal, and
motion-related features. If a weighted model is used, its weights may be
constrained so that:

\[ w_r + w_t + w_m = 1 \]

This equation is only a conceptual example. The features must have
defined meanings and valid normalisation; weights and thresholds should
be learned or selected using suitable data and then evaluated on
held-out subjects and conditions. A score must not be described as a
medical risk probability unless it has been calibrated and validated for
that specific interpretation.

### Development stages

-   **Stage 1 --- Signal processing:** filtering, presence checks,
    respiration extraction, thermal ROI selection, and data-quality
    indicators.
-   **Stage 2 --- Interpretable baseline:** compare simple rules and
    conventional models using clearly defined features.
-   **Stage 3 --- Evaluation:** assess generalisation, false alarms,
    missed detections, calibration, and performance under movement and
    environmental variation.
-   **Stage 4 --- Advanced models:** evaluate time-series approaches
    such as CNNs or LSTMs only if the dataset size and evaluation
    results justify their added complexity.

A more complex model is not automatically a better model. Performance,
interpretability, robustness, and computational cost should guide model
selection.

## Hardware concept

The proposed kiosk-style enclosure is modular and intended for a
controlled, one-person sensing zone.

  -----------------------------------------------------------------------
  Subassembly             Purpose                 Candidate design
                                                  direction
  ----------------------- ----------------------- -----------------------
  Sensor head housing     Protect and align       ABS/polycarbonate or
                          sensing modules         another suitable
                                                  enclosure material

  mmWave radar module     Capture motion-related  Select module based on
                          radar data              supported output,
                                                  range, and software
                                                  access

  Thermal sensor          Capture thermal frames  Select resolution,
                          or measurements         field of view,
                                                  interface, and
                                                  calibration
                                                  requirements

  Main processing unit    Run acquisition,        Embedded computer /
                          processing, and UI      single-board computer
                                                  sized to the workload

  Touch display           Present status and      Display sized for clear
                          measurements            viewing and accessible
                                                  interaction

  Vertical column         Support sensor head and Aluminium extrusion or
                          display                 a suitable structural
                                                  assembly

  Internal brackets       Maintain sensor and     Aluminium or steel,
                          display alignment       subject to load and
                                                  fabrication needs

  Base and feet           Resist tipping and      Weighted base with
                          protect the floor       non-slip feet; verify
                                                  stability by
                                                  calculation and test

  Power subsystem         Supply electronics      Rated supply,
                          safely                  protection, cable
                                                  management, and thermal
                                                  allowance
  -----------------------------------------------------------------------

**These are candidate materials and architectural choices, not final
manufacturing specifications.** Exact components, dimensions, mass,
centre of gravity, power draw, thermal limits, and safety margins should
be established after component selection and engineering validation.

## User interface concept

The interface may display:

-   Radar and thermal sensor connection status
-   Presence / positioning status
-   Estimated respiration rate, when the signal is valid
-   Radar signal-quality and motion indicators
-   Thermal observation and ROI validity
-   Overall measurement status
-   A prompt to repeat the measurement when quality is insufficient
-   Timestamp and anonymised test-session identifier, where appropriate

Avoid displaying a definitive "healthy" or "diseased" result. A usable
prototype should distinguish a successful measurement from a clinical
interpretation.

## Implementation roadmap

-   [ ] **1. Define requirements:** intended setting, sensing distance,
    observation duration, target hardware, and safe operating
    conditions.
-   [ ] **2. Select sensors:** confirm radar data access, thermal output
    format, field of view, interfaces, and vendor documentation.
-   [ ] **3. Build the sensor bench:** acquire raw radar and thermal
    data independently.
-   [ ] **4. Implement quality checks:** detect missing data, movement
    interference, invalid ROI, and sensor faults.
-   [ ] **5. Develop signal processing:** extract respiratory-motion
    features and thermal summaries.
-   [ ] **6. Establish a baseline:** document algorithms, parameters,
    and limitations before adding machine learning.
-   [ ] **7. Build the enclosure:** CAD, mounting, cable routing,
    ventilation, power access, and stability review.
-   [ ] **8. Integrate the UI:** show sensor state, quality, features,
    and repeat-measurement prompts.
-   [ ] **9. Collect suitable data:** follow consent, privacy, and
    institutional requirements; define ground-truth procedures before
    data collection.
-   [ ] **10. Validate:** evaluate repeatability, error, false alarms,
    environmental sensitivity, and failure handling.
-   [ ] **11. Document results:** report measured results separately
    from design goals and simulations.

## Validation plan

Validation should begin with controlled bench tests before any clinical
or real-world screening claims are considered.

  -----------------------------------------------------------------------
  Test                                What to evaluate
  ----------------------------------- -----------------------------------
  Stationary target / controlled      Whether periodic respiratory motion
  breathing                           can be extracted consistently

  Reference respiration comparison    Error and agreement against a
                                      suitable reference instrument

  Controlled movement                 How often movement corrupts
                                      estimates and whether quality
                                      checks catch it

  Distance and orientation variation  Operating envelope and
                                      repeatability

  Thermal reference comparison        Sensor response, calibration,
                                      drift, and environmental
                                      sensitivity

  Ambient-condition variation         Changes caused by temperature,
                                      airflow, and surrounding surfaces

  Repeated measurements               Within-session and between-session
                                      consistency

  Fault injection                     Behaviour when a sensor
                                      disconnects, data freezes, or
                                      readings become invalid

  Mechanical stability                Tip resistance, column deflection,
                                      mounting rigidity, and cable strain

  Power and thermal testing           Actual consumption, component
                                      temperatures, and safe operating
                                      limits
  -----------------------------------------------------------------------

Any human-subject testing should follow applicable institutional review,
informed-consent, privacy, and safety requirements. A prototype should
not be used to make consequential medical decisions without the
necessary evidence, approvals, and clinical oversight.

## Performance metrics

Define evaluation metrics before testing and report measured values with
the test conditions.

-   **Respiration estimation:** mean absolute error (MAE), bias, limits
    of agreement, and valid-estimate rate against a suitable reference.
-   **Thermal measurement:** error against a suitable reference,
    repeatability, drift, and sensitivity to distance and ambient
    conditions.
-   **Measurement quality:** invalid-reading detection rate and the
    proportion of observations requiring a repeat.
-   **Screening model (if developed):** sensitivity, specificity,
    false-positive rate, false-negative rate, calibration, and
    confidence intervals on held-out data.
-   **System performance:** end-to-end measurement time,
    successful-session rate, latency, power consumption, and operating
    temperature.
-   **Mechanical performance:** stability, deflection, mounting
    repeatability, and durability.

Do not report target values as achieved results. Any numerical result
should include the dataset, sample size, protocol, reference method, and
limitations.

## Safety, privacy, and limitations

VITA is intended as an engineering prototype for contactless sensing
research. Key limitations include:

-   Radar-based respiration estimation can fail under gross movement,
    poor orientation, multipath, or weak signal conditions.
-   Thermal surface measurements are influenced by environmental and
    sensor factors and do not directly establish core body temperature.
-   Sensor fusion cannot compensate for systematically inaccurate or
    unrepresentative data.
-   A machine-learning model may perform differently across populations,
    environments, clothing, distances, and sensor configurations.
-   A confidence score is meaningful only when its definition and
    calibration have been evaluated.
-   The prototype is not a substitute for professional assessment,
    validated diagnostic equipment, or emergency medical care.

### Privacy-by-design recommendations

-   Collect only the data necessary for the stated engineering
    objective.
-   Use anonymised or pseudonymous session identifiers.
-   Obtain appropriate consent and approvals for human-subject data
    collection.
-   Restrict access to raw sensor data and define a retention/deletion
    policy.
-   Secure data at rest and in transit where network storage is used.
-   Document what is processed locally and what, if anything, is
    transmitted.

## Potential applications

Subject to validation and appropriate approvals, the architecture could
be explored in:

-   Emergency-care workflow research
-   Temporary healthcare and disaster-relief settings
-   Contactless physiological-sensing research
-   Remote observation and assisted monitoring prototypes
-   Human--machine interface and sensor-fusion demonstrations

These are potential areas for investigation, not claims of proven
clinical suitability.

## Repository structure

A suggested structure for the software repository:

``` text
vita/
├── README.md
├── docs/
│   ├── system-architecture.md
│   ├── hardware-selection.md
│   ├── validation-plan.md
│   └── safety-and-privacy.md
├── hardware/
│   ├── cad/
│   ├── electronics/
│   └── bom/
├── software/
│   ├── acquisition/
│   ├── signal_processing/
│   ├── sensor_fusion/
│   ├── interface/
│   └── tests/
├── data/
│   └── README.md
└── results/
    └── README.md
```

This is a proposed layout; adapt it to the actual files and tools used
in the project. Avoid committing private, identifiable, or
consent-restricted sensor data.

## Getting started

The exact setup depends on the selected radar, thermal sensor,
processing unit, and software stack. Before installation:

1.  Record the exact sensor models and firmware versions.
2.  Install the vendors' supported drivers and SDKs.
3.  Verify that each sensor can be read independently.
4.  Confirm timestamps and units for all outputs.
5.  Run the signal-processing pipeline on recorded test data before
    connecting the full kiosk.
6.  Document installation commands and tested software versions in this
    README as they become known.

No universal install command is provided yet because the hardware and
software dependencies have not been specified.

## Team and contributions

VITA is a modular engineering concept that can involve mechanical
design, electronics, embedded software, signal processing, machine
learning, user-interface design, and validation.

Contributions should include a clear description of the change, relevant
test evidence, known limitations, and any changes to hardware or
software requirements. Keep experimental findings reproducible by
recording configurations and dataset versions.

## Disclaimer

VITA is a prototype concept for research and engineering development. It
is **not a certified medical device**, and its outputs are not intended
to diagnose, treat, cure, or prevent any disease or medical condition.
Contactless radar and thermal observations alone cannot establish a
person's overall health status. Any future clinical use would require
appropriate validation, regulatory review, risk management, and
professional oversight.

------------------------------------------------------------------------

::: {align="center"}
**VITA --- Intelligent contactless screening when rapid attention
matters.**

*Sense responsibly. Validate rigorously. Keep people at the centre.*
:::
