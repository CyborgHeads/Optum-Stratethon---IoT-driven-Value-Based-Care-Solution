# IoMT FHIR Connector for IoT-driven Value-Based Healthcare Solutions

[![Build Status](https://microsofthealthoss.visualstudio.com/FhirServer/_apis/build/status/IoMT/IoMT%20CI%20Build?branchName=main)](https://microsofthealthoss.visualstudio.com/FhirServer/_build/latest?definitionId=12&branchName=main)

The IoMT FHIR Connector for Azure is an open-source project for ingesting data from IoMT (internet of medical things) devices and persisting the data in a FHIR&reg; server. This has been put to use for ingesting the data from the IoT device acquired from the patient and normalize it into a common format. Later this formatted data will be acquired by various applications like Power BI which will later be used for visualization and data compilation.

The IoMT FHIR Connector for Azure is intended only for use in transferring and formatting data.  It is not intended for use as a medical device or to perform any analysis or any medical function, and the performance of the software for such purposes has not been established.  You bear sole responsibility for any use of this software, including incorporation into any product intended for a medical purpose.

The IoMT FHIR Connector for Azure is built with extensibility in mind, enabling developers to modify and extend the capabilities to support additional device mapping template types and FHIR resources. The different points for extension are:
* Normalization: Device data information is extracted into a common format for further processing.
* FHIR Conversion: Normalized and grouped data is mapped to FHIR.  Observations are created or updated according to configured templates and linked to the device and patient.

The IoMT FHIR Connector for Azure empowers developers – saving time when they need to quickly integrate IoMT data into their FHIR server for use in their own applications or providing them with a foundation on which they can customize their own IoMT FHIR connector service. As an open source project, contributions and feedback from the FHIR developer community will continue to improve this project.

# Setup and Requirements
- R4 FHIR server with support for Device, Patient, and Observation resources.
- OAuth 2.0 identity provider with configured [client credentials](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) granted access to the FHIR server.
- An existing device resource and patient resource on the FHIR server. The device should be linked to the patient. Please note the identity extracted for the device during the normalization step should be a device identifier not the internal id.
- A device content template uploaded to the template storage container. See [Configuration](./docs/Configuration.md) for more information.
- A FHIR mapping template uploaded to the template storage container. See [Configuration](./docs/Configuration.md) for more information.

# Getting Started
To get started, there are a few options:
1. Deploy the [IoMT FHIR Connector for Azure](./docs/ARMInstallation.md) by itself for use with Azure API for FHIR and Azure Active directory in the same subscription. Requests made to the Azure API for FHIR will be authenticated using the Azure Active Directory within this subscription - External identity providers cannot be used if the IoMT FHIR Connector for Azure is deployed with this template.
2. Start with a complete [sandbox environment](./docs/Sandbox.md) that includes an instance of [IoT Central](https://azure.microsoft.com/en-us/services/iot-central/) with simulated devices and a deployed instance of the [Azure API for FHIR](https://docs.microsoft.com/en-us/azure/healthcare-apis/).

To send messages to the connector you can [send events](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send) directly to the `devicedata` EventHub deployed as part of IoMT FHIR Connector for Azure or [send events](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-sdks) to one of the Azure IoT solutions and [export messages](./docs/Iot.md) to the connector. 

# Architecture

![alt text](./images/processflow.png "Process Flow")

* **Ingest**: The ingestion point for device data is an Event Hub. [Scale](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-faq#throughput-units) your Event Hub throughput units based on your message volume.
* **Normalize**: Device data is processed and compared to templates defined in the `devicecontent.json` configuration file.  Types, values, and other important information are extracted.  The output is written to a second Event Hub.
* **Group**: Normalized data is grouped according to device identity, measurement type, and the configured time period.  The time period controls the latency that observations are written to FHIR.
* **Transform**: Output from the group and buffering stage is processed.  Observations are created by matching the types from the grouped normalized data to the templates defined in the `fhirmapping.json` configuration file. It is at this point that the device is retrieved from the FHIR server along with the associated patient.  

    **Note** all identity look ups are cached once resolved to decrease load on the FHIR server.  If you plan on reusing devices with multiple patients it is advised you create a *virtual device* resource that is specific to the patient and the virtual device identifier is what is sent in the message payload. The virtual device can be linked to the actual device resource as a parent.
* **Persist**: Once the observation is generated in the FHIR conversion step it is created or merged in the configured destination FHIR server.

## Azure Architecture
![alt text](./images/iomtfhirconnectorazurearchitecture.png "Azure Architecture")

# Documentation
- [Configuration](./docs/Configuration.md): Documents the different configurations required for the connector.
- [Open Source Deployment using Managed Identity](./docs/ARMInstallation.md): Describes how to deploy the IoMT FHIR Connector for Azure using Azure API for FHIR and Azure Active directory in the same subscription.
- [Sandbox Deployment](./docs/Sandbox.md): Describes how to deploy an end to end sandbox environment using IoT Central, IoMT FHIR Connector for Azure, and the Azure API for FHIR.
- [Connecting to Azure IoT](./docs/Iot.md): Describes how to connect the IoMT FHIR Connector for Azure with different Azure IoT solutions like IoT Hub and IoT Central.
- [Debugging](./docs/Debugging.md): Documents steps for local and cloud debugging.

## IoMT Connector Data Mapper

The IoMT Connector Data Mapper is a tool to visualize the mapping configuration for normalizing the device input data and transform it to the FHIR resources. Developers can use this tool to edit and test the mappings, device mapping and FHIR mapping, and export them for uploading to the IoT Connector in the Azure portal. The tool also gives tutorials for developers to understand the mapping configuration.
[Click here for additional details](./tools/data-mapper/)
