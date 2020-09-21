---
title: Provision simulated X.509 device to Azure IoT Hub using C and OpenSSL-generated self-signed certificate
description: This quickstart uses individual enrollments. In this quickstart, you create and provision a simulated X.509 device using C device SDK for Azure IoT Hub Device Provisioning Service (DPS) and OpenSSL to generate a self-signed X509 certificate.
author: smcd253
ms.author: spmcdono
ms.date: 09/21/2020
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps 
ms.custom: mvc
#Customer intent: As a new IoT developer, I want simulate a X.509 device using the C SDK so that I can learn how secure provisioning works with a self-signed X509 certificate.
---

Notes from implementing a self-signed x509 cert on the azure-iot-sdk-c using the provided Provision Device Client Sample

Method 1: Using the provided cert generator… quick-create-simulated-device-x509

Method 2: Generating a self-signed cert using OpenSSL and the C-SDK's Custom HSM
Note: We will be following along the quick-create-simulated-device-x509 tutorial for all of the DPS-related items
	1) Create an azure portal account, an IoT Hub, and a Device Provisioning Service (DPS)
	2) Generate a self-signed cert using OpenSSL
	```bash
	openssl genrsa -out private_key.pem 2048
	# Common Name = global.azure-devices-provisioning.net
	# all other metadata does not matter
	openssl req -new -key private_key.pem -out cert_sign_req.csr
	openssl x509 -req -days 365 -in cert_sign_req.csr -signkey private_key.pem -out X509testcert.pem
	```
	ref <https://stackoverflow.com/questions/14464441/how-to-create-a-self-signed-x509-certificate-with-both-private-and-public-keys> 
	3) Create an individual enrollment under your DPS and upload your X509testcert.pem as the primary certificate.
		a. Be sure to link your IoT hub here before saving
	4) Follow these steps to set up the client to connect to your DPS
	5) Before building the client project, we must build the custom_hsm_example projct to give the client project access to the cert
		a. Open provisioning_client\samples\custom_hsm_example\custom_hsm_example.c
		b. Copy and paste your global device endpoint (usually global.azure-devices-provisioning.net) into the string `COMMON_NAME`
		c. Copy and paste your self-signed certificate X509testcert.pem into the string `CERTIFICATE`
		d. Copy and paste your private key private_key.pem into the string `PRIVATE_KEY`
		e. If you haven't already done so, create a cmake folder under your azuire-iot-sdk-c folder `mkdir cmake`
		f. Now create cmake/hsm `mkdir cmake/hsm`
		g. `cd cmake/hsm`
		h. Now you will build the certificate project `cmake ..\..\provisioning_client\samples\custom_hsm_example`
	6) Now you can configure and build prov_dev_client_sample
		a. Open provisioning_client\samples\prov_dev_client_sample\prov_dev_client_sample.c
		b. Set `global_prov_uri = "global.azure-devices-provisioning.net"` 
		c. Find your ID Scope under your global device endpoint on the DPS overview and set `id_scope = "<your_dps_id_scope>"`
	7) Now you can build and run your project
		a. `cd ..`
		b. `cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=<path_to_azure-iot-sdk-c>\azure-iot-sdk-c\cmake\hsm\Debug\custom_hsm_example.lib ..`
		c. Ex: `cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=C:\Users\crazy_iot_guy\iot_projects\azure-iot-sdk-c\cmake\hsm\Debug\custom_hsm_example.lib ..`
		d. If you haven't yet, download and install Visual Studio 2019
		e. Now open azure-iot-sdk-c/cmake/azure_iot_sdks.sln in Visual Studio
		f. Go to the Solution Explorer on the right of Visual Studio, right click Provision_Samples/prov_dev_client_sample, and click "Set as Startup Project"
		g. Now at the top of Visual Studio you should see a tab titled "Debug." Click this and then click "Start Without Debugging" or just hit ctrl + F5.

You should now see a new Visual Studio command line terminal pop up displaying the following information:
```bash
Registering Device

Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

Registration Information received from service: gettingstarted.azure-devices.net, deviceId: test-dpcs-cert-device
Press enter key to exit:
```
Now go to your IoT Hub on your Azure portal, and on the left side you should see IoT Devices under "Explorers". Click this and you should see a new device with the Device ID "test-dpcs-cert-device" and Authentication Type "SelfSigned"

You have now just automatically provisioned a virtual IoT device using a self-signed certificate and Azure's Device Provisioning Service!

Credits: Jelani Brandon, Ricardo Minguez Pablos (Rido) 

		
		
		
