AWS Chalice Workshop - Media Query Application - AWS Lambda, Chalice, DynamoDB, S3, CloudFront, Rekognition

https://chalice-workshop.readthedocs.io/en/latest/media-query/index.html

Navigation

- Prerequisite: Setting up your environment
- Todo Application
- Media Query Application
- Part 0: Introduction to AWS Lambda and Chalice
- Part 1: Introduction to Amazon Rekognition
- Install the AWS CLI
- Detect image labels using Rekognition
- Part 2: Build a Chalice application using Rekognition
- Part 3: Integrate with a DynamoDB table
- Part 4: Add S3 event source
- Part 5: Add S3 delete event handler
- Part 6: Add REST API to query media files
- Part 7: Add workflow to process videos
- Cleaning up the Chalice application

- Part 0: Introduction to AWS Lambda and Chalice
  - Create a virtualenv and install Chalice
  - Create a new Chalice application
  - Hello world Lambda function
  - Lambda function using event parameter
  - Delete the Chalice application
- Part 1: Introduction to Amazon Rekognition
  - Install the AWS CLI
  - Detect image labels using Rekognition
- Part 2: Build a Chalice application using Rekognition
  - Create a new Chalice project
  - Copy over boilerplate files
  - Write a Lambda function for detecting labels
  - Create a S3 bucket
  - Deploy the Chalice application
- Part 3: Integrate with a DynamoDB table
  - Copy over boilerplate files
  - Create a DynamoDB table
  - Integrate the DynamoDB table
  - Redeploy the Chalice application
- Part 4: Add S3 event source
  - Add Lambda event source for S3 object creation event
  - Redeploy the Chalice application
- Part 5: Add S3 delete event handler
  - Add Lambda function for S3 object deletion
  - Redeploy the Chalice application
- Part 6: Add REST API to query media files
  - Add route for listing media items
  - Add route for retrieving a single media item
  - Redeploy the Chalice application
- Part 7: Add workflow to process videos
  - Introduction to Rekognition object detection in videos
  - Create SNS topic and IAM role
  - Deploy a lambda function for retrieving processed video labels
  - Automate video workflow on S3 uploads and deletions
  - Final Code
- Cleaning up the Chalice application
  - Instructions
  - Validation

https://chalice-workshop.readthedocs.io/en/latest/media-query/01-intro-rekognition.html

# Part 1: Introduction to Amazon Rekognition

The application being built will leverage
`Amazon Rekognition <https://aws.amazon.com/rekognition/>`\_\_ to detect objects
in images and videos. This part of the tutorial will teach you more about
Rekognition and how to detect objects with its API.

## Install the AWS CLI

To interact with the Rekognition API, the AWS CLI will need to be installed.

Instructions

```
1. Check to see the CLI is installed::

    $ aws --version
    aws-cli/1.15.60 Python/3.6.5 Darwin/15.6.0 botocore/1.10.59

   The version of the CLI must be version 1.15.60 or greater.
   We recommend using AWS CLI v2.

2a. If the CLI is not installed, follow the installation instructions in the
    :ref:`aws-cli-setup` section.

2b. If your current CLI version is older than the minimum required version,
    follow the upgrade instructions in the `user guide
    <https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html>`__ to
    upgrade to the latest version of the AWS CLI.


Verification
```

1. Run the following command::

   $ aws --version
   aws-cli/1.15.60 Python/3.6.1 Darwin/15.6.0 botocore/1.10.59

   The version displayed of the CLI must be version 1.15.60 or greater.

## Detect image labels using Rekognition

Use the Rekognition API via the AWS CLI to detect labels in an image.

Instructions

```

1. If you have not already done so, clone the repository for this workshop::

    $ git clone https://github.com/aws-samples/chalice-workshop.git

2. Use the ``detect-labels`` command to detect labels on a sample image::

    $ aws rekognition detect-labels \
        --image-bytes fileb://chalice-workshop/code/media-query/final/assets/sample.jpg


Verification
```

The output of the `detect-labels` command should be::

    {
        "Labels": [
            {
                "Confidence": 85.75711822509766,
                "Name": "Animal"
            },
            {
                "Confidence": 85.75711822509766,
                "Name": "Canine"
            },
            {
                "Confidence": 85.75711822509766,
                "Name": "Dog"
            },
            {
                "Confidence": 85.75711822509766,
                "Name": "German Shepherd"
            },
            {
                "Confidence": 85.75711822509766,
                "Name": "Mammal"
            },
            {
                "Confidence": 85.75711822509766,
                "Name": "Pet"
            },
            {
                "Confidence": 84.56783294677734,
                "Name": "Collie"
            }
        ]
    }

https://chalice-workshop.readthedocs.io/en/latest/media-query/02-chalice-with-rekognition.html

# Part 2: Build a Chalice application using Rekognition

For this part of the tutorial, we will begin writing the media query Chalice
application and integrate Rekognition into the application. This initial
version of the application will accept the S3 bucket and key name of an image,
call the `DetectLabels` API on that stored image, and return the labels
detected for that image. So assuming the `sample.jpg` image is stored in a
bucket `some-bucket` under the key `sample.jpg`, we will be able to invoke
a Lambda function that return the labels Rekognition detected::

    $ echo '{"Bucket": "some-bucket", "Key": "sample.jpg"}' | chalice invoke --name detect_labels_on_image
    ["Animal", "Canine", "Dog", "German Shepherd", "Mammal", "Pet", "Collie"]

For this section, we will be doing the following to create this version of the
application:

.. contents::
:local:
:depth: 1

## Create a new Chalice project

Create the new Chalice project for the Media Query application.

Instructions

```

1. Create a new Chalice project called ``media-query`` with the ``new-project``
   command::

       $ chalice new-project media-query


Verification
```

To ensure that the project was created, list the contents of the newly created
`media-query` directory::

    $ ls media-query
    app.py           requirements.txt

It should contain an `app.py` file and a `requirements.txt` file.

## Copy over boilerplate files

Copy over starting files to facilitate development of the application

Instructions

```


1. Copy over the starting point code for section ``02-chalice-with-rekognition``
   into your ``media-query`` directory::

    $ cp -r chalice-workshop/code/media-query/02-chalice-with-rekognition/. media-query/

   .. note::

      If you are ever stuck and want to skip to the beginning of a different
      part of this tutorial, you can do this by running the same command
      as above, but instead use the ``code`` directory name of the part you
      want to skip to. For example, if you wanted to skip to the beginning of
      Part 5 of this tutorial, you can run the following command with
      ``media-query`` as the current working directory and be ready to start
      Part 5::

       media-query$  cp -r ../chalice-workshop/code/media-query/05-s3-delete-event/. ./


Verification
```

1. Ensure the structure of the `media-query` directory is the
   following::

   $ tree -a media-query
   â”œâ”€â”€ .chalice
   â”‚Â Â â”œâ”€â”€ config.json
   â”‚Â Â â””â”€â”€ policy-dev.json
   â”œâ”€â”€ .gitignore
   â”œâ”€â”€ app.py
   â”œâ”€â”€ chalicelib
   â”‚Â Â â”œâ”€â”€ **init**.py
   â”‚Â Â â””â”€â”€ rekognition.py
   â”œâ”€â”€ recordresources.py
   â”œâ”€â”€ requirements.txt
   â””â”€â”€ resources.json

   For the files that got added, they will be used later in the tutorial but
   for a brief overview of the new files:

   - `chalicelib`: A directory for managing Python modules outside of the
     `app.py`. It is common to put the lower-level logic in the
     `chalicelib` directory and keep the higher level logic in the
     `app.py` file so it stays readable and small. You can read more
     about `chalicelib` in the Chalice
     `documentation <http://chalice.readthedocs.io/en/latest/topics/multifile.html>`\_\_.

   - `chalicelib/rekognition.py`: A utility module to further simplify
     `boto3` client calls to Amazon Rekognition.

   - `.chalice/config.json`: Manages configuration of the Chalice
     application. You can read more about the configuration file in
     the Chalice `documentation <https://chalice.readthedocs.io/en/latest/topics/configfile.html>`\_\_.

   - `.chalice/policy-dev.json`: The IAM policy to apply to your Lambda
     function. This essentially manages the AWS permissions of your
     application

   - `resources.json`: A CloudFormation template with additional resources
     to deploy outside of the Chalice application.

   - `recordresources.py`: Records resource values from the additional
     resources deployed to your CloudFormation stack and saves them
     as environment variables in your Chalice application .

## Write a Lambda function for detecting labels

Fill out the `app.py` file to write a Lambda function that detects labels
on an image stored in a S3 bucket.

Instructions

```

1. Move into the ``media-query`` directory::

    $ cd media-query


2. Add ``boto3``, the AWS SDK for Python, as a dependency in the
   ``requirements.txt`` file:

.. literalinclude:: ../../../code/media-query/03-add-db/requirements.txt
   :linenos:


3. Open the ``app.py`` file and delete all lines of code underneath
   the line: ``app = Chalice(app_name='media-query')``. Your ``app.py`` file
   should only consist of the following lines::

    from chalice import Chalice

    app = Chalice(app_name='media-query')


3. Import ``boto3`` and the ``chalicelib.rekognition`` module in your
   ``app.py`` file:

.. literalinclude:: ../../../code/media-query/03-add-db/app.py
   :linenos:
   :lines: 1-3
   :emphasize-lines: 1,3

4. Add a helper function for instantiating a Rekognition client:

.. literalinclude:: ../../../code/media-query/03-add-db/app.py
   :linenos:
   :lines: 1-15
   :emphasize-lines: 7,10-15

5. Add a new function ``detect_labels_on_image`` decorated by the
   ``app.lambda_function`` decorator. Have the function use a rekognition
   client to detect and return labels on an image stored in a S3 bucket:

.. literalinclude:: ../../../code/media-query/03-add-db/app.py
   :linenos:
   :emphasize-lines: 18-22


Verification
```

1. Ensure the contents of the `requirements.txt` file is:

.. literalinclude:: ../../../code/media-query/03-add-db/requirements.txt
:linenos:

1. Ensure the contents of the `app.py` file is:

.. literalinclude:: ../../../code/media-query/03-add-db/app.py
:linenos:

## Create a S3 bucket

Create a S3 bucket for uploading images and use with the Chalice application.

Instructions

```

1. Use the AWS CLI and the ``resources.json`` CloudFormation template to deploy
   a CloudFormation stack ``media-query`` that contains a S3 bucket::

    $ aws cloudformation deploy --template-file resources.json --stack-name media-query


Verification
```

1. Retrieve and store the name of the S3 bucket using the AWS CLI::

   $ MEDIA_BUCKET_NAME=$(aws cloudformation describe-stacks --stack-name media-query --query "Stacks[0].Outputs[?OutputKey=='MediaBucketName'].OutputValue" --output text)

2. Ensure you can access the S3 bucket by listing its contents::

   $ aws s3 ls $MEDIA_BUCKET_NAME

   Note that the bucket should be empty.

## Deploy the Chalice application

Deploy the chalice application.

Instructions

```

1. Install the dependencies of the Chalice application::

    $ pip install -r requirements.txt


2. Run ``chalice deploy`` to deploy the application::

    $ chalice deploy
    Creating deployment package.
    Creating IAM role: media-query-dev-detect_labels_on_image
    Creating lambda function: media-query-dev-detect_labels_on_image
    Resources deployed:
      - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-detect_labels_on_image

Verification
```

1. Upload the sample workshop image to the S3 bucket::

   $ aws s3 cp ../chalice-workshop/code/media-query/final/assets/sample.jpg s3://$MEDIA_BUCKET_NAME

2. Create a `sample-event.json` file to use with `chalice invoke`::

   $ echo "{\"Bucket\": \"$MEDIA_BUCKET_NAME\", \"Key\": \"sample.jpg\"}" > sample-event.json

3. Run `chalice invoke` on the `detect_labels_on_image` Lambda function::

   $ chalice invoke --name detect_labels_on_image < sample-event.json

   It should return the following labels in the output::

   ["Animal", "Canine", "Dog", "German Shepherd", "Mammal", "Pet", "Collie"]

https://chalice-workshop.readthedocs.io/en/latest/media-query/03-add-db.html

# Part 3: Integrate with a DynamoDB table

Now that we have a Lambda function that can detect labels in an image, let's
integrate a DynamoDB table so we can query information across the various
images stored in our bucket. So instead of returning the labels, the Chalice
application will store the items in a DynamoDB table.

For this section, we will be doing the following to integrate the DynamoDB
table:

.. contents::
:local:
:depth: 1

## Copy over boilerplate files

Copy over files needed for integrating the DynamoDB table into the application

Instructions

```

1. Using ``media-query`` as the current working directory, copy the ``db.py``
   module into the ``chalicelib`` package::

    $ cp ../chalice-workshop/code/media-query/03-add-db/chalicelib/db.py chalicelib/


2. Using ``media-query`` as the current working directory, copy over an updated
   version of the ``resources.json`` file::

    $ cp ../chalice-workshop/code/media-query/03-add-db/resources.json .


Verification
```

1. Ensure the structure of the `media-query` directory includes the
   following files and directories::

   $ tree -a .
   â”œâ”€â”€ .chalice
   â”‚Â Â â”œâ”€â”€ config.json
   â”‚Â Â â””â”€â”€ policy-dev.json
   â”œâ”€â”€ .gitignore
   â”œâ”€â”€ app.py
   â”œâ”€â”€ chalicelib
   â”‚Â Â â”œâ”€â”€ **init**.py
   â”‚Â Â â”œâ”€â”€ db.py
   â”‚Â Â â””â”€â”€ rekognition.py
   â”œâ”€â”€ recordresources.py
   â”œâ”€â”€ requirements.txt
   â””â”€â”€ resources.json

   Note there will be more files listed with `tree` assuming you already
   deployed the application once. However, the files listed from the `tree`
   output above are required.

2. Ensure the contents of the `resources.json` is now the following::

   $ cat resources.json
   {
   "Outputs": {
   "MediaBucketName": {
   "Value": {
   "Ref": "MediaBucket"
   }
   },
   "MediaTableName": {
   "Value": {
   "Ref": "MediaTable"
   }
   }
   },
   "Resources": {
   "MediaBucket": {
   "Type": "AWS::S3::Bucket"
   },
   "MediaTable": {
   "Properties": {
   "AttributeDefinitions": [
   {
   "AttributeName": "name",
   "AttributeType": "S"
   }
   ],
   "KeySchema": [
   {
   "AttributeName": "name",
   "KeyType": "HASH"
   }
   ],
   "ProvisionedThroughput": {
   "ReadCapacityUnits": 5,
   "WriteCapacityUnits": 5
   }
   },
   "Type": "AWS::DynamoDB::Table"
   }
   }
   }

## Create a DynamoDB table

Create a DynamoDB table to store and query information about images in the S3
bucket.

Instructions

```

1. Use the AWS CLI and the ``resources.json`` CloudFormation template to
   redeploy the ``media-query`` CloudFormation stack and create a new
   DynamoDB ::

    $ aws cloudformation deploy --template-file resources.json --stack-name media-query


Verification
```

1. Retrieve and store the name of the DynamoDB table using the AWS CLI::

   $ MEDIA_TABLE_NAME=$(aws cloudformation describe-stacks --stack-name media-query --query "Stacks[0].Outputs[?OutputKey=='MediaTableName'].OutputValue" --output text)

2. Ensure the existence of the table using the `describe-table` CLI command::

   $ aws dynamodb describe-table --table-name $MEDIA_TABLE_NAME
   {
   "Table": {
   "AttributeDefinitions": [
   {
   "AttributeName": "name",
   "AttributeType": "S"
   }
   ],
   "TableName": "media-query-MediaTable-10QEPR0O8DOT4",
   "KeySchema": [
   {
   "AttributeName": "name",
   "KeyType": "HASH"
   }
   ],
   "TableStatus": "ACTIVE",
   "CreationDateTime": 1531769158.804,
   "ProvisionedThroughput": {
   "NumberOfDecreasesToday": 0,
   "ReadCapacityUnits": 5,
   "WriteCapacityUnits": 5
   },
   "TableSizeBytes": 0,
   "ItemCount": 0,
   "TableArn": "arn:aws:dynamodb:us-west-2:123456789123:table/media-query-MediaTable-10QEPR0O8DOT4",
   "TableId": "00eebe92-d59d-40a2-b5fa-32e16b571cdc"
   }
   }

## Integrate the DynamoDB table

Integrate the newly created DynamoDB table into the Chalice application.

Instructions

```

1. Save the DynamoDB table name as an environment variable in the Chalice
   application by running the ``recordresources.py`` script::

    $ python recordresources.py --stack-name media-query


2. Import ``os`` and the ``chalicelib.db`` module in your ``app.py`` file:

.. literalinclude:: ../../../code/media-query/04-s3-event/app.py
   :linenos:
   :lines: 1-6
   :emphasize-lines: 1,5

3. Add a helper function for instantiating a ``db.DynamoMediaDB`` class using
   the DynamoDB table name stored as the environment variable
   ``MEDIA_TABLE_NAME``:

.. literalinclude:: ../../../code/media-query/04-s3-event/app.py
   :linenos:
   :lines: 1-20
   :emphasize-lines: 10,14-20

4. Update the ``detect_labels_on_image`` Lambda function to save the image
   along with the detected labels to the database:

   .. literalinclude:: ../../../code/media-query/04-s3-event/app.py
      :lines: 31-36
      :emphasize-lines: 5-6


Verification
```

1. Ensure the contents of the `config.json` contains environment variables
   for `MEDIA_TABLE_NAME`::

   $ cat .chalice/config.json
   {
   "version": "2.0",
   "app_name": "media-query",
   "stages": {
   "dev": {
   "api_gateway_stage": "api",
   "autogen_policy": false,
   "environment_variables": {
   "MEDIA_TABLE_NAME": "media-query-MediaTable-10QEPR0O8DOT4",
   "MEDIA_BUCKET_NAME": "media-query-mediabucket-fb8oddjbslv1"
   }
   }
   }
   }

   Note that the `MEDIA_BUCKET_NAME` will be present as well in the
   environment variables. It will be used in the next part of the tutorial.

2. Ensure the contents of the `app.py` file is:

.. literalinclude:: ../../../code/media-query/04-s3-event/app.py
:linenos:

## Redeploy the Chalice application

Deploy the updated Chalice application.

Instructions

```

1. Run ``chalice deploy``::

    $ chalice deploy
    Creating deployment package.
    Updating policy for IAM role: media-query-dev-detect_labels_on_image
    Updating lambda function: media-query-dev-detect_labels_on_image
    Resources deployed:
      - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-detect_labels_on_image

Verification
```

1. Run `chalice invoke` with the `sample-event.json` on the updated
   `detect_labels_on_image` Lambda function::

   $ chalice invoke --name detect_labels_on_image < sample-event.json
   null

2. Use the `get-item` CLI command to ensure the `sample.jpg` data
   was populated in the DynamoDB table::

   $ aws dynamodb get-item --table-name $MEDIA_TABLE_NAME \
    --key '{"name": {"S": "sample.jpg"}}'
   {
   "Item": {
   "name": {
   "S": "sample.jpg"
   },
   "labels": {
   "L": [
   {
   "S": "Animal"
   },
   {
   "S": "Canine"
   },
   {
   "S": "Dog"
   },
   {
   "S": "German Shepherd"
   },
   {
   "S": "Mammal"
   },
   {
   "S": "Pet"
   },
   {
   "S": "Collie"
   }
   ]
   },
   "type": {
   "S": "image"
   }
   }
   }

https://chalice-workshop.readthedocs.io/en/latest/media-query/04-s3-event.html

# Part 4: Add S3 event source

So far, we have been manually invoking the Lambda function ourselves in order
to detect objects in the image and add the information to our database.
However, we can automate this workflow using Lambda event sources so that
the Lambda function is invoked every time an object is uploaded to the S3
bucket.

For this section, we will be doing the following:

.. contents::
:local:
:depth: 1

## Add Lambda event source for S3 object creation event

Change the Lambda function to be invoked whenever an object is uploaded to
a S3 bucket via the
`on_s3_event decorator <https://chalice.readthedocs.io/en/latest/topics/events.html#s3-events>`\_\_.

Instructions

```

1. In the ``app.py`` file, change the ``detect_labels_on_image`` signature to
   be named ``handle_object_created`` that accepts a single ``event``
   parameter::

    def handle_object_created(event):


2. Update the decorator on ``handle_object_created`` to use the
   ``app.on_s3_event`` decorator instead and have the Lambda function be
   triggered whenever an object is created in the bucket specified by the
   environment variable ``MEDIA_BUCKET_NAME``::

    @app.on_s3_event(bucket=os.environ['MEDIA_BUCKET_NAME'],
                     events=['s3:ObjectCreated:*'])
    def handle_object_created(event):


3. Add the tuple ``_SUPPORTED_IMAGE_EXTENSTIONS`` representing a list of
   supported image extensions:

   .. literalinclude:: ../../../code/media-query/05-s3-delete-event/app.py
      :lines: 12-15


4. Update the ``handle_object_created`` function to use the new ``event``
   argument of type `S3Event <https://chalice.readthedocs.io/en/latest/api.html#S3Event>`__
   and only do object detection and database additions on specific image
   file extensions:

   .. literalinclude:: ../../../code/media-query/05-s3-delete-event/app.py
      :lines: 35-48
      :emphasize-lines: 4-6,8-9,12-14


Validation
~~~~~~~~~~

1. Ensure the contents of the ``app.py`` file is:

.. literalinclude:: ../../../code/media-query/05-s3-delete-event/app.py
   :linenos:

Redeploy the Chalice application
--------------------------------

Deploy the updated Chalice application.

Instructions
```

1. Run `chalice deploy`::

   $ chalice deploy
   Creating deployment package.
   Creating IAM role: media-query-dev-handle_object_created
   Creating lambda function: media-query-dev-handle_object_created
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_created
   Deleting function: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-detect_labels_on_image
   Deleting IAM role: media-query-dev-detect_labels_on_image
   Resources deployed:

   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created

Validation

```

1. Upload the ``othersample.jpg`` image to the S3 bucket::

    $ aws s3 cp ../chalice-workshop/code/media-query/final/assets/othersample.jpg s3://$MEDIA_BUCKET_NAME

2. Use the ``get-item`` CLI command to ensure the ``othersample.jpg`` data
   was automatically populated in the DynamoDB table::

    $ aws dynamodb get-item --table-name $MEDIA_TABLE_NAME \
        --key '{"name": {"S": "othersample.jpg"}}'
    {
        "Item": {
            "name": {
                "S": "othersample.jpg"
            },
            "labels": {
                "L": [
                    {
                        "S": "Human"
                    },
                    {
                        "S": "People"
                    },
                    {
                        "S": "Person"
                    },
                    {
                        "S": "Phone Booth"
                    },
                    {
                        "S": "Bus"
                    },
                    {
                        "S": "Transportation"
                    },
                    {
                        "S": "Vehicle"
                    },
                    {
                        "S": "Man"
                    },
                    {
                        "S": "Face"
                    },
                    {
                        "S": "Leisure Activities"
                    },
                    {
                        "S": "Tourist"
                    },
                    {
                        "S": "Portrait"
                    },
                    {
                        "S": "Crowd"
                    }
                ]
            },
            "type": {
                "S": "image"
            }
        }
    }

   If the item does not appear, try running the ``get-item`` command after
   waiting for ten seconds. Sometimes, it takes a little bit of time for the
   Lambda function to get triggered.

https://chalice-workshop.readthedocs.io/en/latest/media-query/05-s3-delete-event.html

Part 5: Add S3 delete event handler
===================================

Now that we are automatically importing uploaded images to our table, we need
to be able to automatically delete images from our table that get deleted
from our bucket. This can be accomplished by doing the following:

.. contents::
   :local:
   :depth: 1


Add Lambda function for S3 object deletion
------------------------------------------

Add a new Lambda function that is invoked whenever an object is deleted from
the S3 bucket and if it is an image, removes the image from the table.

Instructions
```

1. In the `app.py` file add a new function `handle_object_removed` that
   is triggered whenever an object gets deleted from the bucket and
   deletes the item from table if it is an image:

.. literalinclude:: ../../../code/media-query/06-web-api/app.py
:lines: 42-46

Verification

```

1. Ensure the contents of the ``app.py`` file is:

.. literalinclude:: ../../../code/media-query/06-web-api/app.py
   :linenos:


Redeploy the Chalice application
--------------------------------

Deploy the updated Chalice application with the new Lambda function.

Instructions
```

1. Run `chalice deploy`::

   $ chalice deploy
   Creating IAM role: media-query-dev-handle_object_removed
   Creating lambda function: media-query-dev-handle_object_removed
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_removed
   Resources deployed:

   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_removed

Verification

```

1. Delete the uploaded ``othersample.jpg`` object from the previous part::

    $ aws s3 rm s3://$MEDIA_BUCKET_NAME/othersample.jpg


2. Use the ``scan`` CLI command to ensure the object is no longer in the
   table::

    $ aws dynamodb scan --table-name $MEDIA_TABLE_NAME
    {
        "Items": [
            {
                "name": {
                    "S": "sample.jpg"
                },
                "labels": {
                    "L": [
                        {
                            "S": "Animal"
                        },
                        {
                            "S": "Canine"
                        },
                        {
                            "S": "Dog"
                        },
                        {
                            "S": "German Shepherd"
                        },
                        {
                            "S": "Mammal"
                        },
                        {
                            "S": "Pet"
                        },
                        {
                            "S": "Collie"
                        }
                    ]
                },
                "type": {
                    "S": "image"
                }
            }
        ],
        "Count": 1,
        "ScannedCount": 1,
        "ConsumedCapacity": null
    }


   If the item still appears, try running the ``scan`` command after
   waiting for ten seconds. Sometimes, it takes a little bit of time for the
   Lambda function to get triggered. In the end, the table should only have the
   ``sample.jpg`` item.

https://chalice-workshop.readthedocs.io/en/latest/media-query/06-web-api.html

Part 6: Add REST API to query media files
=========================================

So far we have been querying the image files stored in our table via the AWS
CLI. However, it would be more helpful to have an API on-top of the table
instead of having to query it directly with the AWS CLI. We will now use Amazon
API Gateway integrations with Lambda to create an API for our application.
This API will have two routes:

* ``GET /`` - List all media items in the table. You can supply the query
  string parameters: ``startswith``, ``media-type``, and ``label`` to further
  filter the media items returned in the API call

* ``GET /{name}`` - Retrieve the media item based on the ``name`` of the media
  item.


To create this API, we will perform the following steps:

.. contents::
   :local:
   :depth: 1

Add route for listing media items
---------------------------------

Add an API route ``GET /`` that lists all items in the table and allows
users to query on ``startswith``, ``media-type``, and ``label``.

Instructions
```

1. In the `app.py` file, define the function `list_media_files()` that
   has the route `GET /` using the
   `app.route <https://chalice.readthedocs.io/en/latest/api.html#Chalice.route>`\_\_
   decorator::

   @app.route('/')
   def list_media_files():

2. Inside of the `list_media_files()` function, extract the query string
   parameters from the
   `app.current_request <https://chalice.readthedocs.io/en/latest/api.html#Request>`\_\_
   object and query the database for the media files:

   .. literalinclude:: ../../../code/media-query/07-videos/app.py
   :lines: 50-55,64-75

Verification

```

1. Ensure the contents of the ``app.py`` file is:

.. literalinclude:: ../../../code/media-query/07-videos/app.py
   :linenos:
   :lines: 1-4,6-55,64-84

2. Install `HTTPie <https://httpie.org/>`__ to query the API::

    $ pip install httpie


3. In a different terminal, run ``chalice local`` to run the API as a server
   locally::

    $ chalice local

4. Use HTTPie to query the API for all images::

    $ http 127.0.0.1:8000/
    HTTP/1.1 200 OK
    Content-Length: 126
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 13:59:35 GMT
    Server: BaseHTTP/0.6 Python/3.6.1

    [
        {
            "labels": [
                "Animal",
                "Canine",
                "Dog",
                "German Shepherd",
                "Mammal",
                "Pet",
                "Collie"
            ],
            "name": "sample.jpg",
            "type": "image"
        }
    ]


5. Use HTTPie to query the API using the query string parameter ``label``::

    $ http 127.0.0.1:8000/ label==Dog
    HTTP/1.1 200 OK
    Content-Length: 126
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:01:22 GMT
    Server: BaseHTTP/0.6 Python/3.6.1

    [
        {
            "labels": [
                "Animal",
                "Canine",
                "Dog",
                "German Shepherd",
                "Mammal",
                "Pet",
                "Collie"
            ],
            "name": "sample.jpg",
            "type": "image"
        }
    ]
    $ http 127.0.0.1:8000/ label==Person
    HTTP/1.1 200 OK
    Content-Length: 2
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:01:46 GMT
    Server: BaseHTTP/0.6 Python/3.6.1

    []

   Feel free to test out any of the other query string parameters as well.


Add route for retrieving a single media item
--------------------------------------------

Add an API route ``GET /{name}`` that retrieves a single item in the table
using the ``name`` of the item.

Instructions
```

1. Import `chalice.NotFoundError` in the `app.py` file:

.. literalinclude:: ../../../code/media-query/07-videos/app.py
:linenos:
:lines: 1-7
:emphasize-lines: 5

2. In the `app.py` file, define the function `get_media_file()` decorated
   by `app.route('/{name}')`:

   .. literalinclude:: ../../../code/media-query/07-videos/app.py
   :lines: 58-59

3. Within the `get_media_file()` function, query the media item using the
   `name` parameter and raise a `chalice.NotFoundError` exception when the
   `name` does not exist in the database:

   .. literalinclude:: ../../../code/media-query/07-videos/app.py
   :lines: 58-63

Verification

```

1. Ensure the contents of the ``app.py`` file is:

.. literalinclude:: ../../../code/media-query/07-videos/app.py
   :linenos:
   :lines: 1-84

2. If the local server is not still running, run ``chalice local`` to
   restart the local API server::

    $ chalice local


3. Use HTTPie to query the API for the ``sample.jpg`` image::

    $ http 127.0.0.1:8000/sample.jpg
    HTTP/1.1 200 OK
    Content-Length: 124
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:09:01 GMT
    Server: BaseHTTP/0.6 Python/3.6.1

    {
        "labels": [
            "Animal",
            "Canine",
            "Dog",
            "German Shepherd",
            "Mammal",
            "Pet",
            "Collie"
        ],
        "name": "sample.jpg",
        "type": "image"
    }



4. Use HTTPie to query the API for an image that does not exist::

    $ http 127.0.0.1:8000/noexists.jpg
    HTTP/1.1 404 Not Found
    Content-Length: 90
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:09:34 GMT
    Server: BaseHTTP/0.6 Python/3.6.1

    {
        "Code": "NotFoundError",
        "Message": "NotFoundError: Media file (noexists.jpg) not found"
    }


Redeploy the Chalice application
--------------------------------

Deploy the Chalice application based on the updates.

Instructions
```

1. Run `chalice deploy`::

   $ chalice deploy
   Creating deployment package.
   Updating policy for IAM role: media-query-dev-handle_object_created
   Updating lambda function: media-query-dev-handle_object_created
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_created
   Updating policy for IAM role: media-query-dev-handle_object_removed
   Updating lambda function: media-query-dev-handle_object_removed
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_removed
   Creating IAM role: media-query-dev-api_handler
   Creating lambda function: media-query-dev
   Creating Rest API
   Resources deployed:

   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_removed
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev
   - Rest API URL: https://1lmxgj9bfl.execute-api.us-west-2.amazonaws.com/api/

Verification

```

1. Reupload the ``othersample.jpg`` image using the CLI::

    $ aws s3 cp ../chalice-workshop/code/media-query/final/assets/othersample.jpg s3://$MEDIA_BUCKET_NAME


2. Use HTTPie to query the deployed API for all media items::

    $ http $(chalice url)
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 126
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:14:27 GMT
    Via: 1.1 a3c7cc30af6c8465e695a3c0d44793e0.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: PAkgH2j5G2er_TZwyQOcwGahwNTR8dhEhrCUklcdDuuEBcKOYQ1-Ug==
    X-Amzn-Trace-Id: Root=1-5b4df9c1-89a47758a7a7989e47799a12;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: KLP2SFnTPHcFeqw=
    x-amzn-RequestId: b5e7488a-89cb-11e8-acbf-eda14961f501

    [
        {
            "labels": [
                "Human",
                "People",
                "Person",
                "Phone Booth",
                "Bus",
                "Transportation",
                "Vehicle",
                "Man",
                "Face",
                "Leisure Activities",
                "Tourist",
                "Portrait",
                "Crowd"
            ],
            "name": "othersample.jpg",
            "type": "image"
        },
        {
            "labels": [
                "Animal",
                "Canine",
                "Dog",
                "German Shepherd",
                "Mammal",
                "Pet",
                "Collie"
            ],
            "name": "sample.jpg",
            "type": "image"
        }
    ]


   Note ``chalice url`` just returns the URL of the remotely deployed API.


3. Use HTTPie to test out a couple of the query string parameters::

    $ http $(chalice url) label=='Phone Booth'
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 207
    Content-Type: application/json
    Date: Sun, 22 Jul 2018 07:49:37 GMT
    Via: 1.1 75fd15ce5d9f38e4c444039a1548df96.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: nYpeS8kk_lFklCA7wCkOI0NO1wabDI3jvs3UpHFlsJ-c0nvlXNrvJQ==
    X-Amzn-Trace-Id: Root=1-5b543710-8beb4000395cd60e5688841a;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: Ka2KpF0nvHcF1hg=
    x-amzn-RequestId: c7e9cabf-8d83-11e8-b109-5f2c96dac9da

    [
        {
            "labels": [
                "Human",
                "People",
                "Person",
                "Phone Booth",
                "Bus",
                "Transportation",
                "Vehicle",
                "Man",
                "Face",
                "Leisure Activities",
                "Tourist",
                "Portrait",
                "Crowd"
            ],
            "name": "othersample.jpg",
            "type": "image"
        }
    ]

    $ http $(chalice url) startswith==sample
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 126
    Content-Type: application/json
    Date: Sun, 22 Jul 2018 07:51:03 GMT
    Via: 1.1 53657f22d99084ad547a21392858391b.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: TORlA6wdOff5n4xHUH9ftnXNxFrTmQsSFG18acx7iwKLA_NsUoUoCg==
    X-Amzn-Trace-Id: Root=1-5b543766-912f6e067cb58ddcb6a973de;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: Ka2YEGNvPHcF8SA=
    x-amzn-RequestId: fb25c9e7-8d83-11e8-898d-8da83b49132b

    [
        {
            "labels": [
                "Animal",
                "Canine",
                "Dog",
                "German Shepherd",
                "Mammal",
                "Pet",
                "Collie"
            ],
            "name": "sample.jpg",
            "type": "image"
        }
    ]


4. Use HTTPie to query the deployed API for ``sample.jpg`` image::

    $ http $(chalice url)sample.jpg
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 124
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 14:16:04 GMT
    Via: 1.1 7ca583dd6abc0b0f42b148142a75588a.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: pzkZ0uZvk5e5W-ZV39v2zCCFAmmRJjDMJZ_I9GyDKhg6WEHotrMmnQ==
    X-Amzn-Trace-Id: Root=1-5b4dfa24-69d586d8e94fb75019b42f24;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: KLQFrF3svHcF32Q=
    x-amzn-RequestId: f0a6a6af-89cb-11e8-8420-e7ec8398ed6b

    {
        "labels": [
            "Animal",
            "Canine",
            "Dog",
            "German Shepherd",
            "Mammal",
            "Pet",
            "Collie"
        ],
        "name": "sample.jpg",
        "type": "image"
    }

https://chalice-workshop.readthedocs.io/en/latest/media-query/07-videos.html

Part 7: Add workflow to process videos
======================================

In the final part of this tutorial, we will add the ability to automatically
process videos uploaded to our S3 bucket and add them to our DynamoDB table.

To accomplish this we will be performing the following steps:

.. contents::
   :local:
   :depth: 1


Introduction to Rekognition object detection in videos
------------------------------------------------------

Detecting labels in a video is a different workflow than detecting image labels
when using Rekognition. Specifically, the workflow is asynchronous where you
must initiate a label detection job using the
`StartLabelDetection <https://docs.aws.amazon.com/rekognition/latest/dg/API_StartLabelDetection.html>`__
API and then call `GetLabelDetection <https://docs.aws.amazon.com/rekognition/latest/dg/API_GetLabelDetection.html>`__
once the job is complete to retrieve all of the detected labels. This step will
introduce you to this workflow.

Instructions
```

1. Upload a sample video to the S3 bucket::

   $ aws s3 cp ../chalice-workshop/code/media-query/final/assets/sample.mp4 s3://$MEDIA_BUCKET_NAME

2. Run the `start-label-detection` command with the AWS CLI to start a
   label detection job on the uploaded video and save the `JobId`::

   $ JOB_ID=$(aws rekognition start-label-detection --video S3Object="{Bucket=$MEDIA_BUCKET_NAME,Name=sample.mp4}" --query JobId --output text)

3. Run the `get-label-detection` command until the `JobStatus` field is
   equal to `SUCCEEDED` and retrieve the video labels::

   $ aws rekognition get-label-detection --job-id $JOB_ID

Verification

```

1. Once the ``JobStatus`` field is equal to ``SUCCEEDED``, the output of the
   ``get-label-detection`` command should contain::

    {
        "JobStatus": "SUCCEEDED",
        "VideoMetadata": {
            "Codec": "h264",
            "DurationMillis": 10099,
            "Format": "QuickTime / MOV",
            "FrameRate": 29.707088470458984,
            "FrameHeight": 960,
            "FrameWidth": 540
        },
        "Labels": [
            {
                "Timestamp": 0,
                "Label": {
                    "Name": "Animal",
                    "Confidence": 66.68909454345703
                }
            },
            {
                "Timestamp": 0,
                "Label": {
                    "Name": "Dog",
                    "Confidence": 60.80849838256836
                }
            },
            {
                "Timestamp": 0,
                "Label": {
                    "Name": "Husky",
                    "Confidence": 51.586997985839844
                }
            },
            {
                "Timestamp": 168,
                "Label": {
                    "Name": "Animal",
                    "Confidence": 58.79970169067383
                }
            },
         ...[SHORTENED]...
     }



Create SNS topic and IAM role
-----------------------------

Rekognition ``StartDetectLabels`` also has the option to publish a message to
an SNS topic once the job has completed. This is a much more efficient solution
than constantly polling the ``GetLabelDetection`` API to wait for the labels to
be detected. In this step, we will create an IAM role and SNS topic that
Rekognition can use to publish this message.

Instructions
```

1. Copy the updated version of the `resources.json` CloudFormation template
   containing an IAM role and SNS topic for Rekognition to publish to::

   $ cp ../chalice-workshop/code/media-query/07-videos/resources.json .

2. Deploy the new resources to your CloudFormation stack using the AWS CLI::

   $ aws cloudformation deploy --template-file resources.json \
    --stack-name media-query --capabilities CAPABILITY_IAM

3. Save the SNS topic and IAM role information as environment variables in the
   Chalice application by running the `recordresources.py` script::

   $ python recordresources.py --stack-name media-query

Verification

```

1. Ensure the contents of the ``config.json`` contains the
   environment variables ``VIDEO_TOPIC_NAME``, ``VIDEO_ROLE_ARN``, and
   ``VIDEO_TOPIC_ARN``::

    $ cat .chalice/config.json
    {
      "version": "2.0",
      "app_name": "media-query",
      "stages": {
        "dev": {
          "api_gateway_stage": "api",
          "autogen_policy": false,
          "environment_variables": {
            "MEDIA_TABLE_NAME": "media-query-MediaTable-10QEPR0O8DOT4",
            "MEDIA_BUCKET_NAME": "media-query-mediabucket-fb8oddjbslv1",
            "VIDEO_TOPIC_NAME": "media-query-VideoTopic-KU38EEHIIUV1",
            "VIDEO_ROLE_ARN": "arn:aws:iam::123456789123:role/media-query-VideoRole-1GKK0CA30VCAD",
            "VIDEO_TOPIC_ARN": "arn:aws:sns:us-west-2:123456789123:media-query-VideoTopic-KU38EEHIIUV1"
          }
        }
      }
    }


Deploy a lambda function for retrieving processed video labels
--------------------------------------------------------------

With the new SNS topic, add a new Lambda function that is triggered on
SNS messages to that topic, calls the ``GetDetectionLabel`` API, and adds the
video with the labels into the database.

Instructions
```

1. Import `json` at the top of the `app.py` file:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 1

2. Then, define the function `add_video_file()` that uses the
   `app.on_sns_message <https://chalice.readthedocs.io/en/latest/api.html#Chalice.on_sns_message>`\_\_
   decorator:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 58-59

3. Update the `add_video_file()` function, to process the `event` argument
   of type `SNSEvent <https://chalice.readthedocs.io/en/latest/api.html#SNSEvent>`\_\_
   by retrieving the job ID from the message, retrieve the processed labels
   from Rekognition, and add the video to the database:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 58-65
   :emphasize-lines: 3-8

4. Run `chalice deploy` to deploy the new Lambda function::

   $ chalice deploy
   Creating deployment package.
   Updating policy for IAM role: media-query-dev-handle_object_created
   Updating lambda function: media-query-dev-handle_object_created
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_created
   Updating policy for IAM role: media-query-dev-handle_object_removed
   Updating lambda function: media-query-dev-handle_object_removed
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_removed
   Creating IAM role: media-query-dev-add_video_file
   Creating lambda function: media-query-dev-add_video_file
   Subscribing media-query-dev-add_video_file to SNS topic media-query-VideoTopic-KU38EEHIIUV1
   Updating policy for IAM role: media-query-dev-api_handler
   Updating lambda function: media-query-dev
   Updating rest API
   Resources deployed:

   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_removed
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-add_video_file
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev
   - Rest API URL: https://1lmxgj9bfl.execute-api.us-west-2.amazonaws.com/api/

Verification

```

1. Retrieve the arn of the deployed SNS topic::

    $ VIDEO_TOPIC_ARN=$(aws cloudformation describe-stacks --stack-name media-query --query "Stacks[0].Outputs[?OutputKey=='VideoTopicArn'].OutputValue" --output text)


2. Retrieve the arn of the deployed IAM role::

    $ VIDEO_ROLE_ARN=$(aws cloudformation describe-stacks --stack-name media-query --query "Stacks[0].Outputs[?OutputKey=='VideoRoleArn'].OutputValue" --output text)



3. Run the ``start-label-detection`` command with the AWS CLI to start a
   label detection job on the uploaded video::

    $ aws rekognition start-label-detection \
        --video S3Object="{Bucket=$MEDIA_BUCKET_NAME,Name=sample.mp4}" \
        --notification-channel SNSTopicArn=$VIDEO_TOPIC_ARN,RoleArn=$VIDEO_ROLE_ARN


4. Wait roughly twenty seconds and then use HTTPie to query for the video against the
   application's API::

    $ http $(chalice url)sample.mp4
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 151
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 21:42:12 GMT
    Via: 1.1 aa42484f82c16d99015c599631def20c.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: GpqmQOwnKcaxb2sP2fi-KSs8LCu24Q6ekKV8Oyo6a0HZ7kcnSGMpnQ==
    X-Amzn-Trace-Id: Root=1-5b4e62b4-da9db3b1e4c95470cbc2b160;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: KMRcNHUQvHcFaDQ=
    x-amzn-RequestId: 43c1cb91-8a0a-11e8-af84-8901f225e7d3

    {
        "labels": [
            "Clothing",
            "Bird Nest",
            "Dog",
            "Human",
            "People",
            "Person",
            "Husky",
            "Animal",
            "Nest",
            "Footwear"
        ],
        "name": "sample.mp4",
        "type": "video"
    }

5. Make sure the ``sample.mp4`` is included when querying for items that have a
   ``video`` media type::

    $ http $(chalice url) media-type==video
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 153
    Content-Type: application/json
    Date: Sun, 22 Jul 2018 07:58:28 GMT
    Via: 1.1 5d53b9570a535c2d94ce93c20abbd471.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: JwvyQ_rEePlEyRAGjtQ1jDnvjXPKt8ea3FiNLdgBbjWnf2G4UTpUaw==
    X-Amzn-Trace-Id: Root=1-5b543923-02ddf1e74491eb77d692c8fd;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: Ka3dkFlHvHcFYIQ=
    x-amzn-RequestId: 0441fc0a-8d85-11e8-b51a-bd624fe1291d

    [
        {
            "labels": [
                "Footwear",
                "Human",
                "People",
                "Nest",
                "Bird Nest",
                "Person",
                "Dog",
                "Husky",
                "Clothing",
                "Animal"
            ],
            "name": "sample.mp4",
            "type": "video"
        }
    ]


Automate video workflow on S3 uploads and deletions
---------------------------------------------------

Now let's update the application so we do not have to manually invoke the
``StartLabelDetection`` API and instead have the API be invoked in
Lambda whenever a video is uploaded to S3. We will also need to automatically
delete the video whenever the video is deleted from S3.

Instructions
```

1. Add the tuple `_SUPPORTED_VIDEO_EXTENSTIONS` representing a list of
   supported video extensions:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 18-22

2. Update the `handle_object_created` function to start a video label
   detection job for videos uploaded to the S3 bucket and have the completion
   notification be published to the SNS topic:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 42-50,105-113
   :emphasize-lines: 6-7,10-11,14-18

3. Update the `handle_object_removed` function to delete items from the
   table that are videos as well:

   .. literalinclude:: ../../../code/media-query/final/app.py
   :lines: 51-55
   :emphasize-lines: 4

4. Run `chalice deploy` to deploy the updated Chalice application::

   $ chalice deploy
   Creating deployment package.
   Updating policy for IAM role: media-query-dev-handle_object_created
   Updating lambda function: media-query-dev-handle_object_created
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_created
   Updating policy for IAM role: media-query-dev-handle_object_removed
   Updating lambda function: media-query-dev-handle_object_removed
   Configuring S3 events in bucket media-query-mediabucket-fb8oddjbslv1 to function media-query-dev-handle_object_removed
   Creating IAM role: media-query-dev-add_video_file
   Creating lambda function: media-query-dev-add_video_file
   Subscribing media-query-dev-add_video_file to SNS topic media-query-VideoTopic-KU38EEHIIUV1
   Updating policy for IAM role: media-query-dev-api_handler
   Updating lambda function: media-query-dev
   Updating rest API
   Resources deployed:

   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_removed
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-add_video_file
   - Lambda ARN: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev
   - Rest API URL: https://1lmxgj9bfl.execute-api.us-west-2.amazonaws.com/api/

Verification

```

1. Delete the previously uploaded ``sample.mp4`` from the S3 bucket::

    $ aws s3 rm s3://$MEDIA_BUCKET_NAME/sample.mp4


2. Ensure the ``sample.mp4`` video no longer is queryable from the
   application's API::

    $ http $(chalice url)sample.mp4
    HTTP/1.1 404 Not Found
    Connection: keep-alive
    Content-Length: 88
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 22:06:57 GMT
    Via: 1.1 e93b65cf89966087a2d9723b4713fb37.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: XD7Wr8-zY8cUAEvnSU_ojyvAadTiNatcJXuztSmBta3Kiluvuvf6ug==
    X-Amzn-Trace-Id: Root=1-5b4e6880-c6c366d38f1e906798146b4b;Sampled=0
    X-Cache: Error from cloudfront
    x-amz-apigw-id: KMVEAFEPPHcFieQ=
    x-amzn-RequestId: b7fba401-8a0d-11e8-a7e4-a9e75b4bb382

    {
        "Code": "NotFoundError",
        "Message": "NotFoundError: Media file (sample.mp4) not found"
    }

3. Reupload the ``sample.mp4`` to the S3 bucket::

    $ aws s3 cp ../chalice-workshop/code/media-query/final/assets/sample.mp4 s3://$MEDIA_BUCKET_NAME


4. After waiting roughly 20 seconds, ensure the ``sample.mp4`` video is
   queryable again from the application's API::

    $ http $(chalice url)sample.mp4
    HTTP/1.1 200 OK
    Connection: keep-alive
    Content-Length: 151
    Content-Type: application/json
    Date: Tue, 17 Jul 2018 21:42:12 GMT
    Via: 1.1 aa42484f82c16d99015c599631def20c.cloudfront.net (CloudFront)
    X-Amz-Cf-Id: GpqmQOwnKcaxb2sP2fi-KSs8LCu24Q6ekKV8Oyo6a0HZ7kcnSGMpnQ==
    X-Amzn-Trace-Id: Root=1-5b4e62b4-da9db3b1e4c95470cbc2b160;Sampled=0
    X-Cache: Miss from cloudfront
    x-amz-apigw-id: KMRcNHUQvHcFaDQ=
    x-amzn-RequestId: 43c1cb91-8a0a-11e8-af84-8901f225e7d3

    {
        "labels": [
            "Clothing",
            "Bird Nest",
            "Dog",
            "Human",
            "People",
            "Person",
            "Husky",
            "Animal",
            "Nest",
            "Footwear"
        ],
        "name": "sample.mp4",
        "type": "video"
    }


Final Code
----------
Congratulations! You have now completed this tutorial. Below is the final code
that you should have wrote in the ``app.py`` of your Chalice application:

.. literalinclude:: ../../../code/media-query/final/app.py


Feel free to add your own media files and/or build additional logic on
top of this application. For the complete final application, see the
`GitHub repository <https://github.com/aws-samples/chalice-workshop/tree/master/code/media-query/final>`__

https://chalice-workshop.readthedocs.io/en/latest/media-query/cleanup.html

Cleaning up the Chalice application
===================================

This part of the tutorial provides instructions on how you can clean up your
deployed resources once you are done using this application. This set of
instructions can be completed at any point during the tutorial to clean up the
application.

Instructions
------------


1. Delete the chalice application::

    $ chalice delete
    Deleting Rest API: kyfn3gqcf0
    Deleting function: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev
    Deleting IAM role: media-query-dev-api_handler
    Deleting function: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-add_video_file
    Deleting IAM role: media-query-dev-add_video_file
    Deleting function: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_removed
    Deleting IAM role: media-query-dev-handle_object_removed
    Deleting function: arn:aws:lambda:us-west-2:123456789123:function:media-query-dev-handle_object_created
    Deleting IAM role: media-query-dev-handle_object_created

2. Delete all objects in your S3 bucket::

    $ aws s3 rm s3://$MEDIA_BUCKET_NAME --recursive
    delete: s3://media-query-mediabucket-4b1h8anboxpa/sample.jpg
    delete: s3://media-query-mediabucket-4b1h8anboxpa/sample.mp4

3. Delete the CloudFormation stack containing the additional AWS resources::

    $ aws cloudformation delete-stack --stack-name media-query

Validation
----------

1. Ensure that the API for the application no longer exists::

    $ chalice url
    Error: Could not find a record of a Rest API in chalice stage: 'dev'


2. Check the existence of a couple of resources from the CloudFormation stack
   to make sure the resources no longer exist::

    $ aws s3 ls s3://$MEDIA_BUCKET_NAME
    An error occurred (NoSuchBucket) when calling the ListObjects operation: The specified bucket does not exist

    $ aws dynamodb describe-table --table-name $MEDIA_TABLE_NAME
    An error occurred (ResourceNotFoundException) when calling the DescribeTable operation: Requested resource not found: Table: media-query-MediaTable-YIM7BMEIOF8Y not found
```
