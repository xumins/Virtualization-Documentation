---
title: Programmatic Installation of Containers
description: Programmatic Installation of Containers
keywords: docker, containers
author: taylorb
ms.date: 04/7/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 50ff9edd-aeb2-4eea-bbea-0d6ee8002e0a
---

This doc is a work in progress and not yet compleate.

# Programmatic Installation of Containers


## Quering the feature using DISM APIs
The containers feature can be installed/uninstalled using a number of tools in Windows including PowerShell, DISM.exe and other UI based tools.  Underlying all of those is the component based servicing infrastructure in Windows which you can utilize to query for the presence of the container feature and whether or not it has been installed.
[DISM API Referance](https://msdn.microsoft.com/en-us/library/windows/desktop/hh825834(v=vs.85).aspx)

This is an adaptation of the [DISM API Package and Feature Functions Sample](https://msdn.microsoft.com/en-us/library/windows/desktop/hh824804(v=vs.85).aspx)
```cpp
#include "windows.h"
#include <stdio.h>
#include "DismApi.h"

int _cdecl wmain()
{
	HRESULT hr = S_OK;
	HRESULT hrLocal = S_OK;
	DismSession session = DISM_SESSION_DEFAULT;
	DismFeatureInfo *pFeatureInfo = NULL;
	UINT uiCount = 0;

	// Initialize the API
	hr = DismInitialize(DismLogErrorsWarningsInfo, L"C:\\MyLogFile.txt", NULL);
	if (FAILED(hr))
	{
		wprintf(L"DismInitialize Failed: %x\n", hr);
		goto Cleanup;
	}

	// Open a session against the mounted image
	hr = DismOpenSession(DISM_ONLINE_IMAGE,
			NULL,
			NULL,
			&session);
	if (FAILED(hr))
	{
		wprintf(L"DismOpenSession Failed: %x\n", hr);
		goto Cleanup;
	}

	// Get the container feature from DISM
	hr = DismGetFeatureInfo(session,
		L"Containers",
		NULL,
		DismPackageName,
		&pFeatureInfo);

	if (FAILED(hr))
	{
		wprintf(L"DismGetFeatureInfo Failed: %x\n", hr);
		goto Cleanup;
	}

	// Possible feature states from https://msdn.microsoft.com/en-us/library/windows/desktop/hh824765(v=vs.85).aspx
	switch (pFeatureInfo->FeatureState)
	{
	case 0:
		wprintf(L"The  %s feature is not present.\n", pFeatureInfo->DisplayName);
		break;
	case 1:
		wprintf(L"An uninstall process for the  %s feature is pending. Additional processes are pending and must be completed before the  %s feature is successfully uninstalled.\n", pFeatureInfo->DisplayName);
		break;
	case 2:
		wprintf(L"The %s feature is staged.\n", pFeatureInfo->DisplayName);
		break;
	case 3:
		wprintf(L"Metadata about the  %s feature has been added to the system, but the  %s feature is not present.\n", pFeatureInfo->DisplayName);
		break;
	case 4:
		wprintf(L"The  %s feature is installed.\n", pFeatureInfo->DisplayName);
		break;
	case 5:
		wprintf(L"The install process for the  %s feature is pending. Additional processes are pending and must be completed before the  %s feature is successfully installed.\n", pFeatureInfo->DisplayName);
		break;
	case 6:
		wprintf(L"The  %s feature has been superseded by a more recent  %s feature.\n", pFeatureInfo->DisplayName);
		break;
	case 7:
		wprintf(L"The  %s feature is partially installed. Some parts of the  %s feature have not been installed.\n", pFeatureInfo->DisplayName);
		break;
	default:
		wprintf(L"Unexpected result\n");
		break;
	}


Cleanup:
	// Delete the memory associated with the objects that were returned

	hrLocal = DismDelete(pFeatureInfo);
	if (FAILED(hrLocal))
	{
		wprintf(L"DismDelete Failed: %x\n", hrLocal);
	}

	// Close the DismSession to free up resources tied to this image servicing session
	hrLocal = DismCloseSession(session);
	if (FAILED(hrLocal))
	{
		wprintf(L"DismCloseSession Failed: %x\n", hrLocal);
	}


	// Shutdown the DISM API to free up remaining resources
	hrLocal = DismShutdown();
	if (FAILED(hrLocal))
	{
		wprintf(L"DismShutdown Failed: %x\n", hr);
	}

	wprintf(L"Return code is: %x\n", hr);
	return hr;
}
```
