---
description: "Learn more about: IMetaDataEmit::SetParamProps Method"
title: "IMetaDataEmit::SetParamProps Method"
ms.date: "03/30/2017"
api_name:
  - "IMetaDataEmit.SetParamProps"
api_location:
  - "mscoree.dll"
api_type:
  - "COM"
f1_keywords:
  - "IMetaDataEmit::SetParamProps"
  - "SetParamProps method [.NET metadata]"
topic_type:
  - "apiref"
---
# IMetaDataEmit::SetParamProps Method

Sets or changes features of a method parameter that was defined by a prior call to [IMetaDataEmit::DefineParam](imetadataemit-defineparam-method.md).

## Syntax

```cpp
HRESULT SetParamProps (
    [in]  mdParamDef  pd,
    [in]  LPCWSTR     szName,
    [in]  DWORD       dwParamFlags,
    [in]  DWORD       dwCPlusTypeFlag,
    [in]  void const  *pValue,
    [in]  ULONG       cchValue
);
```

## Parameters

 `pd`
 [in] The token for the target parameter.

 `szName`
 [in] The name of the parameter in Unicode.

 `dwParamFlags`
 [in] The flags for the parameter.

 `dwCPlusTypeFlag`
 [in] The ELEMENT_TYPE_* for the constant value.

 `pValue`
 [in] The constant value for the parameter.

 `cchValue`
 [in] The size in (Unicode) characters of `pValue`.

## Requirements

 **Platforms:** See [.NET supported operating systems](https://github.com/dotnet/core/blob/main/os-lifecycle-policy.md).

 **Header:** Cor.h

 **Library:** CorGuids.lib

## See also

- [IMetaDataEmit Interface](imetadataemit-interface.md)
- [IMetaDataEmit2 Interface](imetadataemit2-interface.md)
