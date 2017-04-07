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

[DISM API Referance](https://msdn.microsoft.com/en-us/library/windows/desktop/hh825834(v=vs.85).aspx)

Sample code (based on DISM Sample):

    // Get the container feature if present
    hr = DismGetFeatures(session, 
                         L"Containers",
                         NULL,
                         &DismFeature
                         &uiCount); 

    if( FAILED(hr)) 
    {
        wprintf(L"DismGetFeatures Failed: %x\n", hr); 
        goto Cleanup; 
    }

    // If there are no packages named containers than the featuer
    //    is not present on this machine.
    if (uiCount == 0) 
    {
        wprintf(L"The Containers Feature Is Not Present.");
        goto Cleanup;          
    }
    else
    {
        // The feature was found but could be in a number of states
        //    including installed, not installed etc...
        wprintf(L"The Containers is: %d\n", DismFeature.State);
    }
