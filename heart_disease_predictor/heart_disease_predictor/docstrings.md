# Doc Strings
## 1. configs

### 1.1 conf_script

    paths (str) : data path based on relative dirs
    required_columns (list) : columns of the dataset from specific specifying column names by datatypes up to their names after pre-processing.
    meta_columns (list / str) : list / names of columns to be used in meta modelling
    iqr_thresh (int) : iqr threshold for outlier handling
    train_val_test_split (list) : ratio numbers for [train, validation, test]
    summary_type (list) : list of summary names required

## 2. src_model

### 2.1 identify_null_values

    count_null : Counts null values if there are any in the dataset

    :param dataframe: the dataframe to assess null values from
    :return null_values: the count null values from the dataframe

### 2.2 identify_outliers

    handle_outliers : Identifies and removes outliers in a dataframe

    :param dataframe: The dataframe to be assessed
    :param filt_col: A list of columns to identify outliers
    :param threshold : the threshold of the iqr
    :return dataframe: The dataframe with outliers removed

### 2.3 summary_statistics

    get_mdn_or_md : Retrieves the median or mode of the columns of the passed dataframe
    
    :param dataframe: The dataframe to be assessed
    :param stat_type: The type of desc. stat to return. ("mode" or "median")
    :return stat_list: A list of the dataframe's corresponding column statistic values.
        
    :raises ValueError: If a value other than "mode" or "median" is passed as an arg.

### 2.4 age_and_gender
    
    get_age_group : Retrieves the age group of the entry and appends a new column called age_group

    :param dataframe: The dataframe to be assessed
    :return dataframe: A dataframe with age_group of entries classified and appended as a new column.
        
### 2.5 base_models
    
    base_mode_cv : Retrieves a CrossValidator based on the passed base model.

    :param baseModel (pyspark.ml.classification) : a classification model
    :return crossVal : CrossValidator for the model

    :raises ValueError: If a value other than a classification model is passed

### 2.6 train_test_split

    split_data : Split the dataset into training, validation set and test set.
    Uses a stratified sampling method for equal class split.

    :param dataframe : dataset to split
    :param custom_split (list ; optional) : modify the train-validation-test split ratio
    :return train_set, validation_set, test_set (PySpark dataframes) - percentage split based on the provided values

### 2.7 data_pipeline

    build_data_pipeline : Combines all the stages of the data processing.

    :param appendModels (boolean) : if models to be appended [essential to distinguish pipeline for best model fitting]
    :param models (list) : list of models to be appended
    :returns pipeline (pipeline) : pipeline to fit training data

### 2.8 meta_pipeline
    
    build_meta_pipeline : combines all the stages of the meta features processing.
    
    :param withLabel (boolean) : if labels should be included in the pipeline (fo differentiate model evaluation pipeline, and final predictor pipeline)
    :returns pipeline (pipeline) : pipeline to fit base model results

### 2.9 cross_validation

    grid_search_model : Creates a cross validation object and performs grid search over a set of parameters.

    :param paramGrid : grid of parameters
    :param pipeline : model pipeline
    :returns cv : cross validation object

### 2.10 extract_best_model

    extract_best_mode : extracts the best model from using the CrossValidator

    :param crossVal (CrossValidator) : crossValidator to fit the training data
    :returns best_model (ML model) : the best model to from the crossValidator

### 2.11 evaluate_metrics

    evaluate_metrics : retrieves various metrics for a binary classification problem

    :param testResults (pySpark dataframe) : results after testing
    :param predCol (string) : name of the prediction column
    :param evaluatorType (string) : type of evaluator used (ROC)
    
    :returns accuracy, precision, recall, f1, binaryEval, points (double, double, double, double, double, list) :
        various binary evaluation metrics

## 3. src_utilities

### 3.1 utilities

#### read_csv
    
    read_csv : Reads the path and takes the csv within to process.

    :param path (string) : file path of the .csv file
    :returns dataframe (spark dataframe) : the csv file as a spark dataframe with an inferred schema

#### calculate_age

    calculate_age : calculates the user age based on the birthdate given

    :param birth_date (date) : date given by the user
    :returns age (int) : returns age based on the date given and the date today
    
#### predict

    predict : will predict a boolean result based on user input
    
    :param preprocessor (PipelineModel) : preprocessor for feature formatting, vectorization, and normalization
    :param base_model (RandomForestClassifierModel) : base model
    :param meta_preprocessor (PipelineModel) : preprocessor for meta-model
    :param meta_model (LogisticRegressionModel) : meta-model
    :param row (list) : user input as list

### 3.2 datasources_utils

    get_heart_info : Returns heart information
    
    :param: conf (dict): Contains configurations
    :param: handleOutliers (boolean) : handle outliers before returning data
    :param: streamlitPath (boolean) : reference the csv file from the streamlit app directory
    :return: pyspark dataframe: heart_data_info