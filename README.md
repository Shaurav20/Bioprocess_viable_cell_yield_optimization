# Bioprocess Viable Cell Yield Optimization

## Project Overview
This project focuses on optimizing viable cell count (VCC) in bioprocess manufacturing using statistical modeling and machine learning techniques in JMP. The goal is to identify optimal process conditions that maximize cell yield by analyzing experimental data from bioprocess operations.

## Project Structure
```
Bioprocess-Viable-Cell-Yield-Optimization/
│
├── README.md                       # Project overview, methodology, results
├── LICENSE                          # MIT License file
│
├── data/                            # All raw and processed data files
│   ├── raw/
│   │   ├── Bioprocess_data.csv      # Original Kaggle dataset (CSV format)
│   │   └── Bioprocess_data.jmp      # Original Kaggle dataset (JMP format)
│   ├── processed/
│   │   ├── Training_data.jmp        # training split
│   │   ├── Validation_data.jmp      # validation split
│   │   └── Testing_data.jmp         # testing split
│   └── simulation/
│       └── Simulation_data.jmp      # Data used for running simulations/optimization
│
├── scripts/                          # JMP scripts (JSL)
│   ├── Model_3.jsl                   # Final selected model
│   ├── Model_4.jsl                   # Alternative model
│   └── Model_simulation.jsl          # Script to run optimization via prediction profiler
│
├── results/                          # Figures
    ├── ss1.PNG                       # Screenshot 1 (Optimum conditions via prediction profiler)
    └── ss2.PNG                       # Screenshot 2 (Prediction profiler settings)
```

## Dataset
- **Source**: Kaggle - "Bioprocess - Modelling of Titer Production"
- **Target Variable**: Viable Cell Count (VCC)
- **Predictors**: Time, Glucose (Glc), Glutamine (Gln), Ammonia (Amm), Lactate (Lac), temperature, pH, disc_volume_feed, reactor volume, stirring rate, and Experiment ID
- **Data Split**: Training (60%), Validation (30%), Testing (10%)

## Problem-Solving Methodology

### Data Exploration & Quality Assessment
- Generated histograms, normal quantile plots, and outlier box plots for all predictors
- Assessed normality using QQ plots for normal, lognormal, and smooth curve distributions
- Evaluated mean-median distances and identified potential outliers
- Excluded non-important columns from analysis
- Changed **Experiment** column to nominal type

### Correlation & Relationship Analysis
- **Multivariate Correlation Matrix**: Identified significant correlations and multicollinearity
- **Biological Context Consideration**: Some correlations (e.g., time with other variables) were expected due to biological processes
- **Pattern Recognition**:
  - Funnel-shaped relationships: Indicated heteroscedasticity → considered polynomial fitting
  - Triangle-shaped relationships: Kept as-is for modeling
- **Experiment-wise Analysis**: Graph Builder visualizations revealed gradual batch variations without systematic drift

### Statistical Testing & Decision Making
- **ANOVA on Experiments**: p < 0.05 indicated significant differences
- **Tukey HSD Pairwise Comparisons**: Mixed overlapping/non-overlapping mean diamonds
- Decided to include **Experiment_ID as Random Effect** in mixed models to account for batch variability but couldn't find any such addition option.

### Polynomial Degree Determination
For each predictor, systematically tested linear through quartic polynomial fits:

- **Time Analysis**
  - Linear fit: R² = 0.076 (poor), significant lack of fit
  - Cubic polynomial: R² = 0.72 (excellent improvement)
  - Quartic polynomial: R² = 0.73 (minimal improvement)
  - **Decision**: Use cubic polynomial for time (sigmoidal growth pattern)
  - Residual Analysis: No patterns in residuals, near-normal distribution

- **Glucose Analysis**
  - Linear fit: R² = 0.125, funnel-shaped residuals (heteroscedasticity)
  - Higher-degree polynomials: No significant R² improvement
  - **Decision**: Keep as-is with potential transformation

- **Other Predictors**
  - **Amm**: Good fit with 4th degree polynomial
  - **Lac & disc_Glc_feed**: Good fit with 2nd degree polynomial
  - **disc_Gln_feed**: Good fit with 3rd degree polynomial
  - **Reactor volume**: No significant correlation

### Model Development
Built and compared four different models:

- **Model 1**: VCC ~ All variables + interactions + polynomials  
  *Result*: Poor performance, not pursued further

- **Model 2**: VCC ~ Glc + Gln + Glc×Gln + Time³ + Temp + pH + Disc_Volume_Feed + Reactor_Volume

- **Model 3**: log(VCC) ~ Glc + Gln + Glc×Gln + Time³ + Temp + pH + Disc_Volume_Feed + Reactor_Volume  
  *Selected as final model*: Best performance with transformed response

- **Model 4**: VCC ~ Amm + Lac + Amm⁴ + Lac² + Amm×Lac + Time³ + Temp + pH + Disc_Volume_Feed + Reactor_Volume  
  *Alternative metabolic byproducts model*

Saved the **Model_3** and **Model_4** JSL scripts.

### Model Validation & Testing
- Applied **Model 3** to validation and test sets
- Achieved good predictive performance
- **Residual Analysis**: Confirmed model assumptions were met
- **Cross-validation**: Demonstrated generalizability beyond training data

### Optimization via Prediction Profiler
- Launched Profiler from model report
- Set desirability function to **Maximize VCC**
- Automated optimization: JMP calculated factor values for maximum yield

#### Optimal Conditions Identified:
- **Glucose**: 220.93
- **Glutamine**: 0
- **Time**: 130.50
- **Temperature**: 36.56
- **pH**: 6.94
- **Reactor volume**: 0.90
- **Feed volume**: 0.04
- **Desirability score**: Close to 1.0

Saved as **Model_simulation.jsl** script.

### Model Verification
- Applied simulation script to training data
- Generated predicted yield column

#### Visual Verification:
- Fit Y by X plot (Actual vs Predicted) showed points clustered around diagonal

#### Numerical Verification:
- Calculated residuals: **Mean ≈ 0**
- Residual standard deviation ≈ Model RMSE
- R² close to training R²
- Confirmed model accurately represents training relationships

Saved the screenshots (at maximum desirability setting) as **ss1.png** and **ss2.png**.

## Key Insights
- **Time relationship** is cubic, representing sigmoidal cell growth patterns
- **Log transformation** of VCC improved model performance
- **Glucose-Glutamine interaction** is biologically significant (C-N source correlation)
- **Optimal conditions** exist for maximum VCC, identifiable via prediction profiler

## Software Requirements
- **JMP Pro** (Version 17 or higher recommended)
- Basic statistical knowledge for interpretation

## How to Reproduce
1. Load a Kaggle bioprocess-based dataset into JMP. The dataset used in building this model is present here as:
   - `Bioprocess_data.csv`
   - `Bioprocess_data.jmp`
   - `Training_data.jmp`
   - `Validation_data.jmp`
   - `Testing_data.jmp`
   - `Simulation_data.jmp`
2. Preprocess the data according to the previously stated instructions
3. Execute `Model_3.jsl`
4. Use `Model_simulation.jsl` to reproduce optimization

## Future Work
- Incorporate additional process parameters
- Develop real-time monitoring system based on model
- Validate with new experimental data

## License
This project is licensed under the MIT License. You are free to use, modify, and distribute this code with appropriate attribution.

## Author
Shaurav Bhattacharyya

- GitHub: @Shaurav20 (https://github.com/Shaurav20)
- LinkedIn: Shaurav Bhattacharyya (https://www.linkedin.com/in/shaurav-bhattacharyya-a347b5156/)
- Email: shaurav.feb@gmail.com

## Acknowledgements
- **Kaggle** for providing the "Bioprocess - Modelling of Titer Production" dataset
- **JMP** for the statistical discovery software used in this analysis
