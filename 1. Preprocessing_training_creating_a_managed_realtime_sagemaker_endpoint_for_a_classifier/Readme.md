#### ALL Imports in one place in Sagemaker Core

````bash
from sagemaker_core.resources import TrainingJob
from sagemaker_core.shapes import (
    AlgorithmSpecification,
    Channel,
    DataSource,
    S3DataSource,
    ResourceConfig,
    StoppingCondition,
    OutputDataConfig,)

from sagemaker_core.helper.session_helper import Session, get_execution_role

from sagemaker_core.main.shapes import (
    ProcessingInput,
    ProcessingResources,
    AppSpecification,
    ProcessingS3Input,
    ProcessingOutputConfig
)

from sagemaker_core.shapes import (
    ProcessingResources,
    ProcessingClusterConfig,
    ProcessingOutput,
    ProcessingS3Output,
)

from sagemaker_core.resources import ProcessingJob

from sagemaker_core.resources import Model
from sagemaker_core.shapes import ContainerDefinition

from sagemaker_core.resources import Endpoint, EndpointConfig
from sagemaker_core.shapes import ProductionVariant

#initializing sagemaker session
sagemaker_session = Session()
region = sagemaker_session.boto_region_name
role = get_execution_role()
bucket = sagemaker_session.default_bucket()
s3_client = sagemaker_session.boto_session.client('s3')
a= sagemaker_session.upload_data("file_name.py", key_prefix="file_path")
processing_job = ProcessingJob.create(
    processing_job_name=f"sagemaker-core-data-prep-{formatted_timestamp}",
    processing_resources=ProcessingResources(
        cluster_config=ProcessingClusterConfig(
            instance_count=1,
            instance_type="ml.m5.xlarge",
            volume_size_in_gb=20
        )
    app_specification=AppSpecification(
        image_uri=f"683313688378.dkr.ecr.{region}.amazonaws.com/sagemaker-scikit-learn:0.23-1-cpu-py3",
        container_entrypoint=["python3", "/opt/ml/processing/code/preprocess.py"]
    ),

````
#### we can get all the image URI which are in aws sagemaker from this url

Docker Registry Paths and Example Code
Tables containing Amazon ECR registry paths and example code to retrieve the path.
docs.aws.amazon.com

````bash
    processing_inputs=[
        ProcessingInput(
            input_name="input",
            s3_input=ProcessingS3Input(
                s3_uri=f"s3://sagemaker-example-files-prod-{region}/datasets/tabular/synthetic/churn.txt",
                s3_data_type="S3Prefix",
                local_path="/opt/ml/processing/input",
                s3_input_mode="File"
            ),
        ),
````

#### just like input you again mention train, test,val and code for (preprocessing)

````bash
from sagemaker_core.resources import TrainingJob
from sagemaker_core.shapes import (
    AlgorithmSpecification,
    Channel,
    DataSource,
    S3DataSource,
    ResourceConfig,
    StoppingCondition,
    OutputDataConfig,
)
training_job = TrainingJob.create(
    training_job_name=job_name,
    hyper_parameters=hyper_parameters,
    algorithm_specification=AlgorithmSpecification(
        training_image=image, training_input_mode="File"
    ),
````
````bash
output_data_config=OutputDataConfig(s3_output_path=s3_output_path),
resource_config=ResourceConfig(
   instance_type=instance_type,
    instance_count=instance_count,
    volume_size_in_gb=volume_size_in_gb,
 ),
stopping_condition=StoppingCondition(max_runtime_in_seconds=max_runtime_in_seconds),
customer_churn_model = Model.create(
    model_name="customer-churn-xgboost",
    primary_container=ContainerDefinition(image=image, model_data_url=model_s3_uri),
    execution_role_arn=role,
)
model_name = customer_churn_model.get_name() 
endpoint_config = EndpointConfig.create(
    endpoint_config_name=endpoint_config_name,
    production_variants=[
        ProductionVariant(
            variant_name="AllTraffic",
            model_name=model_name,
            instance_type=instance_type,
            initial_instance_count=1,
        )
    ],
)
````
````bash
#### Create the endpoint using the endpoint config
endpoint = Endpoint.create(
    endpoint_name=endpoint_name,
    endpoint_config_name=endpoint_config_name,
)
res = runtime_client.invoke_endpoint(
    EndpointName=endpoint_name,  # Use the endpoint name from earlier
    ContentType="text/csv",
    Body=sample_payload
)
# Delete endpoint
endpoint.delete()

# Delete endpoint configuration
endpoint_config.delete()

# Delete model
customer_churn_model.delete()
````
