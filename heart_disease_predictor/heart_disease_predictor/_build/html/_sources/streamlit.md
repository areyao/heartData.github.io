# Streamlit and Deployment

Streamlit was used to create an interface suitable for user input and real-time predictions.

## What is Streamlit?
Streamlit is a python library used for creating and sharing custom web apps for machine learning and data science. It is a framework that allows users to create web components that cater oto user input and display with only is inbuilt functions. Additionally, it is compatible with most of the major python libraries such as numpy, pandas, matplotlib, and in our case, pyspark.

To install streamlit, type the following code into your working terminal:

    pip install streamlit

As any python library, a suitable import and import name is needed:

    import streamlit as st

Using various streamlit functions, we can now utilize them to create the Heart Disease Risk Prediction app.

### Web App Creation
#### Streamlit Predictor

Usually, streamlit components update the entire application whenever a value is updated. In this case, it may affect the final prediction as values aren't entirely set before predicting.
To remedy this, the use of st.form is highly recommended.

The following block of code constitutes the prediction UI:

    with st.form(key = 'heart_input'):
    # User Age
    today = dt.datetime.today()
    user_birthDate = st.date_input("Birthdate : ", today, min_value= dt.date(today.year - 100, 1, 1), max_value= today)
    user_age = int(calculate_age(user_birthDate))

    # User sex
    sex_assignments = {'Male': 'M', 'Female': 'F'}
    sex_selection = st.selectbox('Select your sex : ', sex_assignments.keys())
    user_sex = sex_assignments[sex_selection]

    # Diabetes- FastingBS > 120
    diabetes_assignments = {'Yes': 1, 'No': 0}
    diabetes_selection = st.selectbox('Do you have Diabetes? : ', diabetes_assignments.keys())
    user_diabetes = int(diabetes_assignments[diabetes_selection])

    # Exercise Induced Angina - ExrIndAng
    exrindang_assignments = {'Yes': 'Y', 'No': 'N'}
    exrindang_selection = st.selectbox('Does your chest pain when you exercise? : ', exrindang_assignments.keys())
    user_ExrIndAng = exrindang_assignments[exrindang_selection]

    # Medically Consulted Portion

    st.write("The following portion must have medically tested information available.")

    # User BloodPressure - Latest systolic blood pressure (The first number) 90-200
    user_bp = int(st.number_input("Latest Systolic Blood Pressure : ", min_value = 80, max_value = 370, format = '%d'))

    # MaxHeartRate
    user_maxhr = int(st.number_input("Maximum Heart Rate : ", min_value=60, max_value=202, format='%d'))

    #Cholesterol
    user_chol = int(st.number_input("Total Cholesterol : ", min_value = 80, max_value = 370, format = '%d'))

    # Consulted Chestpain Type
    chestpain_assignments = {'Typical Angina': 'TA', 'Atypical Angina': 'ATA', 'Non-anginal pain': 'NAP', 'Asymptomatic': 'ASY'}
    chestpain_selection = st.selectbox('Chest Pain : ', chestpain_assignments.keys())
    user_chestpain = chestpain_assignments[chestpain_selection]

    #RestingElectro
    ecg_assignments = {'Normal': 'Normal', 'Having ST-T wave abnormality': 'ST', "Showing probable or definite left ventricular hypertrophy by Estes' criteria" : 'LVH'}
    ecg_selection = st.selectbox('Resting Electrocardiographic Results: ', ecg_assignments.keys())
    user_ecg = ecg_assignments[ecg_selection]

    # Slope
    slope_assignments = {'Upsloping' : 'Up', 'Flat': 'Flat', 'Downsloping': 'Down'}
    slope_selection = st.selectbox('The slope of the peak exercise ST segment : ', slope_assignments.keys())
    user_slope = slope_assignments[slope_selection]

    # Previous Peak
    user_peak = float(st.slider('Previous Peak', min_value=0.0, max_value=10.0))

    if "user_input" not in st.session_state:
        st.session_state["user_input"] = None

    submit_btn = st.form_submit_button('Run Prediction')
    if submit_btn:
        initial_row = [[user_age, user_sex, user_chestpain, user_bp, user_chol, user_diabetes, user_ecg, user_maxhr,
                       user_ExrIndAng, user_peak, user_slope]]
        st.session_state["user_input"] = initial_row
        predict(preprocessor, base_model, meta_preprocessor, meta_model, initial_row)

Once the submit button is pressed, it will return a text based on the prediction:

    # Code snippet from predict method in utilities
    if (int(value) == 1):
        st.error(f'You are at risk for Heart Disease. Please consult a medical professional.')
    else:
        st.success(f'You are not at risk for Heart Disease.')

#### Model Insights Dashboard
Another component of the web app is the Model Insights Dashboard. This includes two figures that display the RandomForestClassifier feature importances:
- One created with Plotly with an interactive interface,
- The other with pythons SHAP library that breaks down the Plotly figure into two class (0 - not at risk, 1 - at risk)

The final figure shows the ROC value of the Logistic Regression model to clearly visualize the model's performance.
(The closer the curve is to the upper left, the more effective it is at predicting an accurate result.)

#### Running the App Locally
To run the app locally, just go to the directory containing it and run this code in the command line.
    
    streamlit run Predictor.py

## What is Docker?
Although streamlit already hosts its own deployment feature, we will be using docker with this project.
Docker lets you build, test, and deploy applications quickly. It is a highly realiable, low-cost way to build, ship, and run distributed application at any scale.
There are two steps in docker deployment:
1. Creating a Docker Image
2. Containerizing a Docker Image


## Creating a Docker Image
### Dockerfile
A Dockerfile is a text document with no file extension that contains a script of instructions that a user could call on the command line to build a container image.
The following Dockerfile was used in order to create an image.

    FROM python:3.9-slim

    WORKDIR /web
    
    COPY . ./
    
    RUN pip3 install -r requirements.txt
    
    ENTRYPOINT ["streamlit", "run", "Predictor.py"]

requirements.txt in this case is a text file that contains all of your python dependencies. This includes all python libraries you have used throughout your project.

    # requirements.txt
    pandas~=2.0.3
    pyspark == 3.4.1
    streamlit == 1.24.1
    numpy == 1.23.1
    matplotlib~=3.7.2
    datetime
    shap == 0.41.0
    plotly == 5.15.0
    spark-submit == 1.4.0
    jupyter~=1.0.0
    pip~=23.1.2
    docutils~=0.18.1
    setuptools~=67.8.0
    packaging~=23.0
    pyparsing~=3.0.9

It is safe to note that these files, including your initial Streamlit .py app is in the same directory location. This will ensure that references are all done correctly.

After setting up both the Dockerfile and the requirements.txt, you can now build the Docker Image.

In the same directory as the Dockerfile, requirements.txt, and the Predictor.py -- run the following code in the terminal.

    docker build -t heartbeat .

This will build an image called "heartbeat". As observed, the creation will take some time as dependencies will need to be installed within the image.

After creating your image, you can check if it exists.

    docker images

We can now use this image to containerize.
## Containerizing a Docker Image

After creating your image, the run the following line of code to create your container.

    docker run -p 8080:8501 heartbeat

After running, you will be shown two links to the containerized app. This will allow you to view your streamlit app in deployment.
