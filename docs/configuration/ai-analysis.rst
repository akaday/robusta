.. _ai-analysis-overview:

AI Analysis
==========================

Why use HolmesGPT?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Robusta can integrate with `Holmes GPT <https://github.com/robusta-dev/holmesgpt>`_ to analyze health issues on your cluster, and to run AI based root cause analysis for alerts.

When available, AI based investigations can be launched in one of two ways:

1. Click the ``Ask Holmes`` button in Slack. The AI investigation will be sent back as a new message.

.. image:: /images/robusta-holmes-investigation.png
    :width: 600px

2. In the Robusta UI, click the ``Root Cause`` tab on an alert.

.. image:: /images/ai-root-causeanalysis.png
    :width: 600px

Configuring HolmesGPT
^^^^^^^^^^^^^^^^^^^^^^

Add ``enableHolmesGPT: true`` to the Robusta Helm values, and then follow these steps:

1. Choose an AI model - we highly recommend using GPT-4o to get the most accurate results! Other models may work, but are not officially supported.
2. :ref:`Configure your AI provider with the chosen model <Choosing and configuring an AI provider>`.
3. :ref:`Optional: Configure HolmesGPT Access to SaaS Data <Configuring HolmesGPT Access to SaaS Data>`.

Choosing and configuring an AI provider
----------------------------------------

Choose an AI provider below and follow the instructions:

.. tab-set::

    .. tab-item:: Robusta AI
        :name: robusta-ai

        Robusta AI is the premium AI service provided by Robusta. It is currently free to use while in beta. To use Robusta AI, you must have a Robusta account and be using the Robusta UI.

        To use Robusta AI, update your helm values (``generated_values.yaml`` file) with the following configuration:

        .. code-block:: yaml

            enableHolmesGPT: true
            holmes:
              additionalEnvVars:
              - name: ROBUSTA_AI
                value: "true"

        Run a :ref:`Helm Upgrade <Simple Upgrade>` to apply the configuration.

    .. tab-item:: OpenAI
        :name: open-ai

        Create a secret with your OpenAI API key:

        .. code-block:: bash

          kubectl create secret generic holmes-secrets --from-literal=openAiKey='<API_KEY_GOES_HERE>'

        Then add the following to your helm values (``generated_values.yaml`` file):

        .. code-block:: yaml

            enableHolmesGPT: true
            holmes:
              additionalEnvVars:
              - name: MODEL
                value: gpt-4o
              - name: OPENAI_API_KEY
                valueFrom:
                  secretKeyRef:
                    name: holmes-secrets
                    key: openAiKey

        Run a :ref:`Helm Upgrade <Simple Upgrade>` to apply the configuration.

    .. tab-item:: Azure AI
        :name: azure-ai

        Go into your Azure portal, **change the default rate-limit to the maximum**, and find the following parameters:

        * API_VERSION
        * DEPLOYMENT_NAME
        * ENDPOINT
        * API_KEY

        .. details:: Step-By-Step Instruction for Azure Portal

          The following steps cover how to obtain the correct AZURE_API_VERSION value and how to increase the token limit to prevent rate limiting.

          1. Go to your Azure portal and choose `Azure OpenAI`

          .. image:: /images/AzureAI/AzureAI_HolmesStep1.png
              :width: 600px

          2. Click your AI service

          .. image:: /images/AzureAI/AzureAI_HolmesStep2.png
              :width: 600px

          3. Click Go to Azure Open AI Studio

          .. image:: /images/AzureAI/AzureAI_HolmesStep3.png
              :width: 600px

          4. Choose Deployments

          .. image:: /images/AzureAI/AzureAI_HolmesStep4.png
              :width: 600px

          5. Select your Deployment - note the DEPLOYMENT_NAME!

          .. image:: /images/AzureAI/AzureAI_HolmesStep5.png
              :width: 600px

          6. Click Open in Playground

          .. image:: /images/AzureAI/AzureAI_HolmesStep6.png
              :width: 600px

          7. Go to View Code

          .. image:: /images/AzureAI/AzureAI_HolmesStep7.png
              :width: 600px

          8. Choose Python and scroll to find the ENDPOINT, API_KEY, and API_VERSION. Copy them! You will need them for Robusta's Helm values.

          .. image:: /images/AzureAI/AzureAI_HolmesStep8.png
              :width: 600px

          9. Go back to Deployments, and click Edit Deployment

          .. image:: /images/AzureAI/AzureAI_HolmesStep9.png
              :width: 600px

          10. MANDATORY: Increase the token limit. Change this value to at least 450K tokens for Holmes to work properly. We recommend choosing the highest value available. (Holmes queries Azure AI infrequently but in bursts. Therefore the overall cost of using Holmes with Azure AI is very low, but you must increase the quota to avoid getting rate-limited on a single burst of requests.)

          .. image:: /images/AzureAI/AzureAI_HolmesStep10.png
              :width: 600px


        Create a secret with the Azure API key you found above:

        .. code-block:: bash

          kubectl create secret generic holmes-secrets --from-literal=azureOpenAiKey='<AZURE_API_KEY_GOES_HERE>'


        Update your helm values (``generated_values.yaml`` file) with the following configuration:

        .. code-block:: yaml

            enableHolmesGPT: true
            holmes:
              additionalEnvVars:
              - name: MODEL
                value: azure/<DEPLOYMENT_NAME>  # replace with deployment name from the portal (e.g. avi-deployment), leave "azure/" prefix
              - name: MODEL_TYPE
                value: gpt-4o                   # your azure deployment model type
              - name: AZURE_API_VERSION
                value: <API_VERSION>            # replace with API version you found in the Azure portal
              - name: AZURE_API_BASE
                value: <AZURE_ENDPOINT>         # fill in the base endpoint url of your azure deployment - e.g. https://my-org.openai.azure.com/
              - name: AZURE_API_KEY
                valueFrom:
                  secretKeyRef:
                    name: holmes-secrets
                    key: azureOpenAiKey

        Run a :ref:`Helm Upgrade <Simple Upgrade>` to apply the configuration.

    .. tab-item:: AWS Bedrock
        :name: aws-bedrock

        You will need the following AWS parameters:

        * BEDROCK_MODEL_NAME
        * AWS_ACCESS_KEY_ID
        * AWS_SECRET_ACCESS_KEY

        Create a secret with your AWS credentials:

        .. code-block:: bash

          kubectl create secret generic holmes-secrets --from-literal=awsAccessKeyId='<YOUR_AWS_ACCESS_KEY_ID>' --from-literal=awsSecretAccessKey'<YOUR_AWS_SECRET_ACCESS_KEY>'

        Update your helm values (``generated_values.yaml`` file) with the following configuration:

        .. code-block:: yaml

            enableHolmesGPT: true
            holmes:
              enablePostProcessing: true
              additionalEnvVars:
              - name: MODEL
                value: bedrock/anthropic.claude-3-5-sonnet-20240620-v1:0  # your bedrock model - replace with your own exact model name
              - name: AWS_REGION_NAME
                value: us-east-1
              - name: AWS_ACCESS_KEY_ID
                valueFrom:
                  secretKeyRef:
                    name: holmes-secrets
                    key: awsAccessKeyId
              - name: AWS_SECRET_ACCESS_KEY
                valueFrom:
                  secretKeyRef:
                    name: holmes-secrets
                    key: awsSecretAccessKey

        Run a :ref:`Helm Upgrade <Simple Upgrade>` to apply the configuration.

Configuring HolmesGPT Access to SaaS Data
----------------------------------------------------

To use HolmesGPT with the Robusta UI, one further step may be necessary, depending on how Robusta is configured.

* If you define the Robusta UI token directly in your Helm values, HolmesGPT can read the token automatically and no further setup is necessary.
* If you store the Robusta UI token in a Kubernetes secret, follow the instructions below.

Note: the same Robusta UI token is used for the Robusta UI sink and for HolmesGPT.

Reading the Robusta UI Token from a secret in HolmesGPT
************************************************************

1. Review your existing Robusta Helm values - you should have an existing section similar to this, which reads the Robusta UI token from a secret:

.. code-block:: yaml

    runner:
      additional_env_vars:
      - name: UI_SINK_TOKEN
        valueFrom:
          secretKeyRef:
            name: my-robusta-secrets
            key: ui-token

    sinksConfig:
    - robusta_sink:
        name: robusta_ui_sink
        token: "{{ env.UI_SINK_TOKEN }}"

2. Add the following to your Helm values, directing HolmesGPT to use the same secret, passed as an environment variable named ``ROBUSTA_UI_TOKEN``:

.. code-block:: yaml

    holmes:
      additionalEnvVars:
      ....
      - name: ROBUSTA_UI_TOKEN
        valueFrom:
          secretKeyRef:
            name: my-robusta-secrets
            key: ui-token

Run a :ref:`Helm Upgrade <Simple Upgrade>` to apply the configuration.

Test Holmes Integration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this section we will see Holmes in action by deploying a crashing pod and analyzing the alert with AI.

Before we proceed, you must follow the instructions above and configure Holmes.

Once everything is setup:

1. Deploy a crashing pod to simulate an issue.

.. code-block:: yaml

    kubectl apply -f https://raw.githubusercontent.com/robusta-dev/kubernetes-demos/main/crashpod/broken.yaml

2. Go to the **Timeline** in `platform.robusta.dev  <https://platform.robusta.dev/>`_ and click on the ``CrashLoopBackOff`` alert

.. image:: /images/AI_Analysis_demo.png
    :width: 1000px

3. Click the "Root Cause" tab on the top. This gives you the result of an investigation done by HolmesGPT based on the alert.

.. image:: /images/AI_Analysis_demo2.png
    :width: 1000px

Additionally your alerts on Slack will have an "Ask Holmes" button that sends an analysis back to Slack.

.. warning::

  Due to technical limitations with Slack, alerts analyzed from Slack will be sent to the AI without alert-labels.

  This means sometimes the AI won't know the namespace, pod name, or other metadata and the results may be less accurate.

  For the most accurate results, it is best to use the Robusta UI.


Advanced - Customizing HolmesGPT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Adding Custom Tools to Holmes
-------------------------------------

Holmes allows you to define custom toolsets that enhance its functionality by enabling additional tools to run Kubernetes commands or other tasks.

In this guide, we will show how to add a custom toolset to Holmes in your ``generated_values.yaml`` file.

.. code-block:: yaml

    enableHolmesGPT: true
    holmes:
      additionalEnvVars:
        - name: ROBUSTA_AI
          value: "true"
      toolsets:
        # Name of the toolset (for example "mycompany/internal-tools")
        # Used for informational purposes only (e.g. to print the name of the toolset if it can't be loaded)
        - name: "resource_explanation"
          # List of tools the LLM can use - this is the important part
          tools:
          # Name is a unique identifier for the tool
            - name: "explain_resource"
              # The LLM looks at this description when deciding what tools are relevant for each task
              description: "Provides detailed explanation of Kubernetes resources using kubectl explain"
              # A templated bash command using Jinja2 templates
              # The LLM can only control parameters that you expose as template variables like {{ resource_name }}
              command: "kubectl explain {{ resource_name }}"


``toolsets``: Defines a custom toolset, in this case, a ``resource_explanation``, which allows Holmes to use the ``kubectl explain`` command to provide details about various Kubernetes resources.

Once you have updated the ``generated_values.yaml`` file, apply the changes by running the Helm upgrade command:

.. code-block:: bash

    helm upgrade robusta robusta/robusta --values=generated_values.yaml --set clusterName=<YOUR_CLUSTER_NAME>

After the deployment, the custom toolset is automatically available for Holmes to use. Holmes will now be able to run the ``kubectl explain`` tool whenever required, allowing it to provide details about various Kubernetes resources.


Adding a tool that requires a new binary
----------------------------------------------

In some cases, adding a new tool to Holmes might require installing additional packages that are not included in the base Holmes Docker image. This guide explains how to create a custom Docker image that includes the new binaries and update your Helm deployment to use the custom image.

As an example, we'll add a new HolmesGPT tool that uses the ``jq`` binary, which isn't present in the original image:

**Example Dockerfile to add jq:**

.. code-block:: bash

    FROM python:3.11-slim

    ENV PYTHONUNBUFFERED=1
    ENV PATH="/venv/bin:$PATH"
    ENV PYTHONPATH=$PYTHONPATH:.:/app/holmes

    WORKDIR /app

    COPY --from=builder /app/venv /venv
    COPY . /app

    # We're installing here libexpat1, to upgrade the package to include a fix to 3 high CVEs. CVE-2024-45491,CVE-2024-45490,CVE-2024-45492
    RUN apt-get update \
        && apt-get install -y \
        git \
        apt-transport-https \
        gnupg2 \
        && apt-get purge -y --auto-remove \
        && apt-get install -y --no-install-recommends libexpat1 \
        && rm -rf /var/lib/apt/lists/*

    # Example of installing jq
    RUN apt-get install -y jq

Now, you will need to **build and push** the Docker image to your container registry.

**Abstracted Instructions for Building and Pushing the Docker Image**:

1. **Build the Docker Image**:
   Depending on the tools and binaries you need, build the custom Docker image with the appropriate tag.

   .. code-block:: bash

       docker build -t <your-registry>/<your-project>/holmes-custom:<tag> .

   Replace:
   - ``<your-registry>``: Your Docker registry (e.g., ``us-central1-docker.pkg.dev`` for Google Artifact Registry).
   - ``<your-project>``: Your project or repository name.
   - ``<tag>``: The desired tag for the image (e.g., ``latest``, ``v1.0``).

2. **Push the Image to Your Registry**:
   After building the image, push it to your container registry:

   .. code-block:: bash

       docker push <your-registry>/<your-project>/holmes-custom:<tag>

   This ensures that the image is available for your Kubernetes deployment.


After pushing your custom Docker image, update your ``generated_values.yaml`` to use this custom image for Holmes.

.. code-block:: yaml

    enableHolmesGPT: true
    holmes:
      registry: <your-registry>/<your-project>  # Use your custom registry
      image: <image>:<tag>  # Specify the image with the tag you used when pushing the image
      additionalEnvVars:
        - name: ROBUSTA_AI
          value: "true"
      toolsets:
        - name: "json_processor"
          prerequisites:
            - command: "jq --version"  # Ensure jq is installed
          tools:
            - name: "process_json"
              description: "A tool that uses jq to process JSON input"
              command: "echo '{{ json_input }}' | jq '.'"  # Example jq command to format JSON


Finally, after updating your ``generated_values.yaml``, apply the changes to your Helm deployment:

.. code-block:: bash

    helm upgrade robusta robusta/robusta --values=generated_values.yaml --set clusterName=<YOUR_CLUSTER_NAME>

This will update the deployment to use the custom Docker image, which includes the new binaries. The ``toolsets`` defined in the configuration will now be available for Holmes to use, including any new binaries like ``jq``.


Adding Permissions for Additional Resources
----------------------------------------------

There are scenarios where HolmesGPT may require access to additional Kubernetes resources or CRDs to perform specific analyses or interact with external tools.

You will need to extend its ClusterRole rules whenever HolmesGPT needs to access resources that are not included in its default configuration.

Common Scenarios for Adding Permissions:

* External Integrations and CRDs: When HolmesGPT needs to access custom resources (CRDs) in your cluster, like ArgoCD Application resources or Istio VirtualService resources.
* Additional Kubernetes resources: By default, Holmes can only access a limited number of Kubernetes resources. For example, Holmes has no access to Kubernetes secrets by default. You can give Holmes access to more built-in cluster resources if it is useful for your use case.

As an example, let's consider a case where we ask HolmesGPT to analyze the state of Argo CD applications and projects to troubleshoot issues related to application deployments managed by Argo CD, but it doesn't have access to the relevant CRDs.

**Steps to Add Permissions for Argo CD:**

1. **Update generated_values.yaml with Required Permissions:**

Add the following configuration under the ``customClusterRoleRules`` section:

.. code-block:: yaml

    enableHolmesGPT: true
    holmes:
      customClusterRoleRules:
        - apiGroups: ["argoproj.io"]
          resources: ["applications", "appprojects"]
          verbs: ["get", "list", "watch"]

2. **Apply the Configuration:**

Deploy the updated configuration using Helm:

.. code-block:: bash

  helm upgrade robusta robusta/robusta --values=generated_values.yaml --set clusterName=<YOUR_CLUSTER_NAME>

This will grant HolmesGPT the necessary permissions to analyze Argo CD applications and projects.
Now you can ask HolmesGPT questions like "What is the current status of all Argo CD applications in the cluster?" and it will be able to answer.
