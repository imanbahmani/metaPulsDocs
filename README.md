# 📚 Meta-Puls API Documentation
# Complete Unreal Engine Integration Guide

**Version:** 1.2.0  
**Base URL:** `https://meta-puls.com/api`  
**Last Updated:** December 31, 2025

---

# 📋 Table of Contents

1. [Setup & Data Structures](#1-setup--data-structures)
2. [Authentication](#2-authentication)
   - 2.1 Login
   - 2.2 Logout
3. [User Profile](#3-user-profile)
4. [TryOn Products](#4-tryon-products)
   - 4.1 Get All Products
   - 4.2 Get Single Product
5. [TryOn List (Favorites)](#5-tryon-list-favorites)
   - 5.1 Get List
   - 5.2 Add to List
   - 5.3 Remove from List
   - 5.4 Clear List
6. [Shopping Cart](#6-shopping-cart)
   - 6.1 Get Cart
   - 6.2 Add to Cart
   - 6.3 Update Cart Item
   - 6.4 Remove from Cart
   - 6.5 Clear Cart
7. [Web Integration](#7-web-integration)

---

# 1. Setup & Data Structures

## 1.1 MetaPulsTypes.h

```cpp
// MetaPulsTypes.h
#pragma once
#include "CoreMinimal.h"
#include "MetaPulsTypes.generated.h"

// =====================================
// PRODUCT (Unreal Engine Format)
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsProduct
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) FString Name;
    UPROPERTY(BlueprintReadOnly) FString Description;
    UPROPERTY(BlueprintReadOnly) FString Category;
    UPROPERTY(BlueprintReadOnly) float Price;
    UPROPERTY(BlueprintReadOnly) FString Currency;
    UPROPERTY(BlueprintReadOnly) FString PriceFormatted;
    
    // Files
    UPROPERTY(BlueprintReadOnly) FString Image;
    UPROPERTY(BlueprintReadOnly) FString ImageUrl;
    UPROPERTY(BlueprintReadOnly) FString Glb;
    UPROPERTY(BlueprintReadOnly) FString GlbUrl;
    UPROPERTY(BlueprintReadOnly) FString Ply;
    UPROPERTY(BlueprintReadOnly) FString PlyUrl;
    
    // 3D Transform
    UPROPERTY(BlueprintReadOnly) FVector Location;
    UPROPERTY(BlueprintReadOnly) FRotator Rotation;
    UPROPERTY(BlueprintReadOnly) FVector Scale;
};

// =====================================
// FURNITURE (Local Position)
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsFurniture
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) int32 FurnitureId;
    UPROPERTY(BlueprintReadOnly) FString Name;
    UPROPERTY(BlueprintReadOnly) FVector Position;  // LOCAL to house
    UPROPERTY(BlueprintReadOnly) FRotator Rotation;
    UPROPERTY(BlueprintReadOnly) FVector Scale;
};

// =====================================
// HOUSE
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsHouse
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) int32 HouseAssetId;
    UPROPERTY(BlueprintReadOnly) FString HouseName;
    UPROPERTY(BlueprintReadOnly) int32 ParcelId;
    UPROPERTY(BlueprintReadOnly) FString ParcelName;
    UPROPERTY(BlueprintReadOnly) FVector Position;  // WORLD position
    UPROPERTY(BlueprintReadOnly) FRotator Rotation;
    UPROPERTY(BlueprintReadOnly) FVector Scale;
    UPROPERTY(BlueprintReadOnly) double GeoLat;
    UPROPERTY(BlueprintReadOnly) double GeoLng;
    UPROPERTY(BlueprintReadOnly) int32 FurnitureCount;
    UPROPERTY(BlueprintReadOnly) TArray<FMetaPulsFurniture> Furniture;
    UPROPERTY(BlueprintReadOnly) FString CreatedAt;
};

// =====================================
// USER
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsUser
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) FString Name;
    UPROPERTY(BlueprintReadOnly) FString Email;
    UPROPERTY(BlueprintReadOnly) FString Avatar;
    UPROPERTY(BlueprintReadOnly) FString WalletAddress;
    UPROPERTY(BlueprintReadOnly) int32 HousesCount;
    UPROPERTY(BlueprintReadOnly) int32 TotalFurniture;
};

// =====================================
// TRYON LIST ITEM
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsTryOnItem
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 ListId;
    UPROPERTY(BlueprintReadOnly) int32 ProductId;
    UPROPERTY(BlueprintReadOnly) FString AddedAt;
    UPROPERTY(BlueprintReadOnly) FMetaPulsProduct Product;
};

// =====================================
// CART ITEM
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsCartItem
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) int32 ProductId;
    UPROPERTY(BlueprintReadOnly) int32 Quantity;
    UPROPERTY(BlueprintReadOnly) FString ProductName;
    UPROPERTY(BlueprintReadOnly) FString ProductSku;
    UPROPERTY(BlueprintReadOnly) float ProductPrice;
    UPROPERTY(BlueprintReadOnly) FString ProductImage;
    UPROPERTY(BlueprintReadOnly) float Subtotal;
    UPROPERTY(BlueprintReadOnly) FString SubtotalFormatted;
};

// =====================================
// CART
// =====================================
USTRUCT(BlueprintType)
struct FMetaPulsCart
{
    GENERATED_BODY()

    UPROPERTY(BlueprintReadOnly) int32 Id;
    UPROPERTY(BlueprintReadOnly) int32 ItemsCount;
    UPROPERTY(BlueprintReadOnly) int32 TotalQuantity;
    UPROPERTY(BlueprintReadOnly) float Total;
    UPROPERTY(BlueprintReadOnly) FString TotalFormatted;
    UPROPERTY(BlueprintReadOnly) FString Currency;
    UPROPERTY(BlueprintReadOnly) TArray<FMetaPulsCartItem> Items;
};
```

## 1.2 Request Headers

| Header | Value | Required |
|--------|-------|----------|
| `Content-Type` | `application/json` | POST/PUT |
| `Accept` | `application/json` | All |
| `Authorization` | `Bearer {token}` | Protected |

---

# 2. Authentication

## 2.1 Login

🔵 **POST** `/api/login`  
**Auth Required:** No

### cURL

```bash
curl -X POST "https://meta-puls.com/api/login" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"email":"imanbahmani@gmail.com","password":"Iman2025!"}'
```

### Request Body

```json
{
    "email": "imanbahmani@gmail.com",
    "password": "Iman2025!"
}
```

### Response (200 OK)

```json
{
    "success": true,
    "token": "7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606",
    "user": {
        "id": 13,
        "name": "imanbahmani",
        "email": "imanbahmani@gmail.com",
        "avatar": null,
        "wallet_address": "0x9287ea1f32316b6dce37c32d48f36a8fa69aed45",
        "created_at": "2025-10-02T19:30:00.000000Z"
    },
    "stats": {
        "houses_count": 3,
        "total_furniture": 8
    },
    "houses": [
        {
            "id": 1,
            "house_asset_id": 5,
            "house_name": "Tiny House",
            "parcel_id": 35,
            "parcel_name": "Parcel #28 - 519m²",
            "position": { "x": -62.6717, "y": 37.603, "z": 0 },
            "rotation": { "x": 0, "y": 0, "z": 0 },
            "scale": { "x": 1, "y": 1, "z": 1 },
            "geo_location": { "lat": 51.15716088, "lng": 6.82466839 },
            "furniture_count": 2,
            "furniture": [
                {
                    "id": 1,
                    "furniture_id": 1,
                    "name": "Modern L-Shaped Sofa",
                    "position": { "x": -168.5432, "y": -258.5541, "z": 0 },
                    "rotation": { "x": 0, "y": 0, "z": 45 },
                    "scale": { "x": 1, "y": 1, "z": 1 }
                }
            ],
            "created_at": "2025-12-29 15:30:29"
        }
    ]
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::Login(const FString& Email, const FString& Password)
{
    // Create HTTP Request
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/login"));
    Request->SetVerb(TEXT("POST"));
    Request->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    
    // Create JSON body
    TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
    JsonObject->SetStringField(TEXT("email"), Email);
    JsonObject->SetStringField(TEXT("password"), Password);
    
    FString RequestBody;
    TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&RequestBody);
    FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);
    
    Request->SetContentAsString(RequestBody);
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnLoginResponse);
    Request->ProcessRequest();
    
    UE_LOG(LogTemp, Log, TEXT("Login request sent for: %s"), *Email);
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnLoginResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        UE_LOG(LogTemp, Error, TEXT("Login request failed"));
        OnLoginComplete.Broadcast(false, TEXT("Connection failed"), FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    int32 ResponseCode = Response->GetResponseCode();
    FString ResponseString = Response->GetContentAsString();
    
    UE_LOG(LogTemp, Log, TEXT("Login Response [%d]: %s"), ResponseCode, *ResponseString);
    
    if (ResponseCode != 200)
    {
        OnLoginComplete.Broadcast(false, TEXT("Invalid credentials"), FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    // Parse JSON
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject) || !JsonObject.IsValid())
    {
        OnLoginComplete.Broadcast(false, TEXT("Invalid JSON response"), FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    // Get token
    AuthToken = JsonObject->GetStringField(TEXT("token"));
    UE_LOG(LogTemp, Log, TEXT("Token received: %s"), *AuthToken);
    
    // Parse user
    FMetaPulsUser User;
    const TSharedPtr<FJsonObject>* UserObject;
    if (JsonObject->TryGetObjectField(TEXT("user"), UserObject))
    {
        User.Id = (*UserObject)->GetIntegerField(TEXT("id"));
        User.Name = (*UserObject)->GetStringField(TEXT("name"));
        User.Email = (*UserObject)->GetStringField(TEXT("email"));
        User.Avatar = (*UserObject)->GetStringField(TEXT("avatar"));
        User.WalletAddress = (*UserObject)->GetStringField(TEXT("wallet_address"));
    }
    
    // Parse stats
    const TSharedPtr<FJsonObject>* StatsObject;
    if (JsonObject->TryGetObjectField(TEXT("stats"), StatsObject))
    {
        User.HousesCount = (*StatsObject)->GetIntegerField(TEXT("houses_count"));
        User.TotalFurniture = (*StatsObject)->GetIntegerField(TEXT("total_furniture"));
    }
    
    // Parse houses
    TArray<FMetaPulsHouse> Houses;
    const TArray<TSharedPtr<FJsonValue>>* HousesArray;
    if (JsonObject->TryGetArrayField(TEXT("houses"), HousesArray))
    {
        for (const TSharedPtr<FJsonValue>& HouseValue : *HousesArray)
        {
            Houses.Add(ParseHouse(HouseValue->AsObject()));
        }
    }
    
    CurrentUser = User;
    UserHouses = Houses;
    
    OnLoginComplete.Broadcast(true, TEXT("Login successful"), User, Houses);
}

// Helper: Parse House
FMetaPulsHouse UMetaPulsAPI::ParseHouse(const TSharedPtr<FJsonObject>& JsonObject)
{
    FMetaPulsHouse House;
    
    House.Id = JsonObject->GetIntegerField(TEXT("id"));
    House.HouseAssetId = JsonObject->GetIntegerField(TEXT("house_asset_id"));
    House.HouseName = JsonObject->GetStringField(TEXT("house_name"));
    House.ParcelId = JsonObject->GetIntegerField(TEXT("parcel_id"));
    House.ParcelName = JsonObject->GetStringField(TEXT("parcel_name"));
    House.FurnitureCount = JsonObject->GetIntegerField(TEXT("furniture_count"));
    House.CreatedAt = JsonObject->GetStringField(TEXT("created_at"));
    
    // Position
    const TSharedPtr<FJsonObject>* PosObj;
    if (JsonObject->TryGetObjectField(TEXT("position"), PosObj))
    {
        House.Position = FVector(
            (*PosObj)->GetNumberField(TEXT("x")),
            (*PosObj)->GetNumberField(TEXT("y")),
            (*PosObj)->GetNumberField(TEXT("z"))
        );
    }
    
    // Rotation
    const TSharedPtr<FJsonObject>* RotObj;
    if (JsonObject->TryGetObjectField(TEXT("rotation"), RotObj))
    {
        House.Rotation = FRotator(
            (*RotObj)->GetNumberField(TEXT("x")),
            (*RotObj)->GetNumberField(TEXT("y")),
            (*RotObj)->GetNumberField(TEXT("z"))
        );
    }
    
    // Scale
    const TSharedPtr<FJsonObject>* ScaleObj;
    if (JsonObject->TryGetObjectField(TEXT("scale"), ScaleObj))
    {
        House.Scale = FVector(
            (*ScaleObj)->GetNumberField(TEXT("x")),
            (*ScaleObj)->GetNumberField(TEXT("y")),
            (*ScaleObj)->GetNumberField(TEXT("z"))
        );
    }
    
    // Geo Location
    const TSharedPtr<FJsonObject>* GeoObj;
    if (JsonObject->TryGetObjectField(TEXT("geo_location"), GeoObj))
    {
        House.GeoLat = (*GeoObj)->GetNumberField(TEXT("lat"));
        House.GeoLng = (*GeoObj)->GetNumberField(TEXT("lng"));
    }
    
    // Furniture
    const TArray<TSharedPtr<FJsonValue>>* FurnitureArray;
    if (JsonObject->TryGetArrayField(TEXT("furniture"), FurnitureArray))
    {
        for (const TSharedPtr<FJsonValue>& FurnValue : *FurnitureArray)
        {
            House.Furniture.Add(ParseFurniture(FurnValue->AsObject()));
        }
    }
    
    return House;
}

// Helper: Parse Furniture
FMetaPulsFurniture UMetaPulsAPI::ParseFurniture(const TSharedPtr<FJsonObject>& JsonObject)
{
    FMetaPulsFurniture Furniture;
    
    Furniture.Id = JsonObject->GetIntegerField(TEXT("id"));
    Furniture.FurnitureId = JsonObject->GetIntegerField(TEXT("furniture_id"));
    Furniture.Name = JsonObject->GetStringField(TEXT("name"));
    
    const TSharedPtr<FJsonObject>* PosObj;
    if (JsonObject->TryGetObjectField(TEXT("position"), PosObj))
    {
        Furniture.Position = FVector(
            (*PosObj)->GetNumberField(TEXT("x")),
            (*PosObj)->GetNumberField(TEXT("y")),
            (*PosObj)->GetNumberField(TEXT("z"))
        );
    }
    
    const TSharedPtr<FJsonObject>* RotObj;
    if (JsonObject->TryGetObjectField(TEXT("rotation"), RotObj))
    {
        Furniture.Rotation = FRotator(
            (*RotObj)->GetNumberField(TEXT("x")),
            (*RotObj)->GetNumberField(TEXT("y")),
            (*RotObj)->GetNumberField(TEXT("z"))
        );
    }
    
    const TSharedPtr<FJsonObject>* ScaleObj;
    if (JsonObject->TryGetObjectField(TEXT("scale"), ScaleObj))
    {
        Furniture.Scale = FVector(
            (*ScaleObj)->GetNumberField(TEXT("x")),
            (*ScaleObj)->GetNumberField(TEXT("y")),
            (*ScaleObj)->GetNumberField(TEXT("z"))
        );
    }
    
    return Furniture;
}
```

> ⚠️ **Important:** Furniture position is LOCAL to house center.  
> World Position = House.Position + Furniture.Position

---

## 2.2 Logout

🔵 **POST** `/api/logout`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X POST "https://meta-puls.com/api/logout" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "Logged out successfully"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::Logout()
{
    if (AuthToken.IsEmpty())
    {
        UE_LOG(LogTemp, Warning, TEXT("Cannot logout: No token"));
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/logout"));
    Request->SetVerb(TEXT("POST"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnLogoutResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnLogoutResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    // Clear local data regardless of response
    AuthToken.Empty();
    CurrentUser = FMetaPulsUser();
    UserHouses.Empty();
    
    if (bSuccess && Response.IsValid() && Response->GetResponseCode() == 200)
    {
        UE_LOG(LogTemp, Log, TEXT("Logout successful"));
        OnLogoutComplete.Broadcast(true, TEXT("Logged out successfully"));
    }
    else
    {
        UE_LOG(LogTemp, Warning, TEXT("Logout request failed, but local data cleared"));
        OnLogoutComplete.Broadcast(true, TEXT("Logged out locally"));
    }
}
```

---

# 3. User Profile

## 3.1 Get Profile

🟢 **GET** `/api/profile`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X GET "https://meta-puls.com/api/profile" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "user": {
        "id": 13,
        "name": "imanbahmani",
        "email": "imanbahmani@gmail.com",
        "avatar": null,
        "wallet_address": "0x9287ea1f32316b6dce37c32d48f36a8fa69aed45"
    },
    "stats": {
        "houses_count": 3,
        "total_furniture": 8
    },
    "houses": [
        {
            "id": 1,
            "house_asset_id": 5,
            "house_name": "Tiny House",
            "parcel_id": 35,
            "parcel_name": "Parcel #28 - 519m²",
            "position": { "x": -62.6717, "y": 37.603, "z": 0 },
            "rotation": { "x": 0, "y": 0, "z": 0 },
            "scale": { "x": 1, "y": 1, "z": 1 },
            "geo_location": { "lat": 51.15716088, "lng": 6.82466839 },
            "furniture_count": 2,
            "furniture": [
                {
                    "id": 1,
                    "furniture_id": 1,
                    "name": "Modern L-Shaped Sofa",
                    "position": { "x": -168.5432, "y": -258.5541, "z": 0 },
                    "rotation": { "x": 0, "y": 0, "z": 45 },
                    "scale": { "x": 1, "y": 1, "z": 1 }
                },
                {
                    "id": 2,
                    "furniture_id": 2,
                    "name": "Classic Leather Sofa",
                    "position": { "x": 162.1522, "y": -301.1694, "z": 0 },
                    "rotation": { "x": 0, "y": 0, "z": 0 },
                    "scale": { "x": 1, "y": 1, "z": 1 }
                }
            ],
            "created_at": "2025-12-29 15:30:29"
        },
        {
            "id": 3,
            "house_asset_id": 5,
            "house_name": "Tiny House",
            "parcel_id": 33477,
            "parcel_name": "Parcel #649 - 51m²",
            "position": { "x": -11.6799, "y": 105.1192, "z": 0 },
            "rotation": { "x": 0, "y": 0, "z": 0 },
            "scale": { "x": 1, "y": 1, "z": 1 },
            "geo_location": { "lat": null, "lng": null },
            "furniture_count": 3,
            "furniture": [
                {
                    "id": 13,
                    "furniture_id": 7,
                    "name": "King Size Bed",
                    "position": { "x": 163.5187, "y": 198.5584, "z": 0 },
                    "rotation": { "x": 0, "y": 0, "z": 0 },
                    "scale": { "x": 1, "y": 1, "z": 1 }
                }
            ],
            "created_at": "2025-12-29 18:45:27"
        },
        {
            "id": 2,
            "house_asset_id": 2,
            "house_name": "Cozy Cottage",
            "parcel_id": 67401,
            "parcel_name": "Parcel #112 - 321m²",
            "position": { "x": -305.7319, "y": -92.7501, "z": 0 },
            "rotation": { "x": 0, "y": 0, "z": 360 },
            "scale": { "x": 1, "y": 1, "z": 1 },
            "geo_location": { "lat": null, "lng": null },
            "furniture_count": 3,
            "furniture": [
                {
                    "id": 7,
                    "furniture_id": 11,
                    "name": "Smart TV 65\"",
                    "position": { "x": 0, "y": 0, "z": 0 },
                    "rotation": { "x": 0, "y": 0, "z": 0 },
                    "scale": { "x": 1, "y": 1, "z": 1 }
                }
            ],
            "created_at": "2025-12-29 16:05:33"
        }
    ]
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::GetProfile()
{
    if (AuthToken.IsEmpty())
    {
        UE_LOG(LogTemp, Error, TEXT("GetProfile: No auth token"));
        OnProfileLoaded.Broadcast(false, FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/profile"));
    Request->SetVerb(TEXT("GET"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnProfileResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnProfileResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid() || Response->GetResponseCode() != 200)
    {
        UE_LOG(LogTemp, Error, TEXT("GetProfile failed"));
        OnProfileLoaded.Broadcast(false, FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        OnProfileLoaded.Broadcast(false, FMetaPulsUser(), TArray<FMetaPulsHouse>());
        return;
    }
    
    // Parse user (same as login)
    FMetaPulsUser User;
    const TSharedPtr<FJsonObject>* UserObject;
    if (JsonObject->TryGetObjectField(TEXT("user"), UserObject))
    {
        User.Id = (*UserObject)->GetIntegerField(TEXT("id"));
        User.Name = (*UserObject)->GetStringField(TEXT("name"));
        User.Email = (*UserObject)->GetStringField(TEXT("email"));
        User.WalletAddress = (*UserObject)->GetStringField(TEXT("wallet_address"));
    }
    
    const TSharedPtr<FJsonObject>* StatsObject;
    if (JsonObject->TryGetObjectField(TEXT("stats"), StatsObject))
    {
        User.HousesCount = (*StatsObject)->GetIntegerField(TEXT("houses_count"));
        User.TotalFurniture = (*StatsObject)->GetIntegerField(TEXT("total_furniture"));
    }
    
    // Parse houses
    TArray<FMetaPulsHouse> Houses;
    const TArray<TSharedPtr<FJsonValue>>* HousesArray;
    if (JsonObject->TryGetArrayField(TEXT("houses"), HousesArray))
    {
        for (const TSharedPtr<FJsonValue>& HouseValue : *HousesArray)
        {
            Houses.Add(ParseHouse(HouseValue->AsObject()));
        }
    }
    
    CurrentUser = User;
    UserHouses = Houses;
    
    UE_LOG(LogTemp, Log, TEXT("Profile loaded: %s with %d houses"), *User.Name, Houses.Num());
    OnProfileLoaded.Broadcast(true, User, Houses);
}
```

---

# 4. TryOn Products

## 4.1 Get All Products

🟢 **GET** `/api/tryon/products`  
**Auth Required:** No (Public)

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter: `upper`, `lower`, `dress` |
| `search` | string | Search by name or SKU |

### cURL

```bash
# All products
curl "https://meta-puls.com/api/tryon/products" \
  -H "Accept: application/json"

# Filter by category
curl "https://meta-puls.com/api/tryon/products?category=dress"

# Search
curl "https://meta-puls.com/api/tryon/products?search=BAG"
```

### Response (200 OK)

```json
{
    "NumberOfProducts": 39,
    "Products": [
        {
            "Id": 30,
            "Name": "GR-1159",
            "Price": 89,
            "Currency": "EUR",
            "PriceFormatted": "€89.00",
            "Image": "GR-1159.jpg",
            "ImageUrl": "https://meta-puls.com/storage/shop-products/GR-1159.jpg",
            "Glb": "GR-1159.glb",
            "GlbUrl": "https://meta-puls.com/storage/shop-products/GR-1159.glb",
            "Ply": "GR-1159.ply",
            "PlyUrl": "https://meta-puls.com/storage/shop-products/GR-1159.ply",
            "LocationX": 0,
            "LocationY": 0,
            "LocationZ": 0,
            "RotationX": 0,
            "RotationY": 0,
            "RotationZ": 0,
            "ScaleX": 1,
            "ScaleY": 1,
            "ScaleZ": 1
        },
        {
            "Id": 2,
            "Name": "BAG-104",
            "Price": 45,
            "Currency": "EUR",
            "PriceFormatted": "€45.00",
            "Image": "BAG-104.jpg",
            "ImageUrl": "https://meta-puls.com/storage/shop-products/BAG-104.jpg",
            "Glb": "BAG-104.glb",
            "GlbUrl": "https://meta-puls.com/storage/shop-products/BAG-104.glb",
            "Ply": "BAG-104.ply",
            "PlyUrl": "https://meta-puls.com/storage/shop-products/BAG-104.ply",
            "LocationX": 0,
            "LocationY": 0,
            "LocationZ": 0,
            "RotationX": -90,
            "RotationY": 0,
            "RotationZ": 90,
            "ScaleX": 0.25,
            "ScaleY": 0.25,
            "ScaleZ": 0.25
        }
    ]
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::GetProducts(const FString& Category, const FString& Search)
{
    FString URL = TEXT("https://meta-puls.com/api/tryon/products");
    
    // Add query parameters
    TArray<FString> Params;
    if (!Category.IsEmpty())
    {
        Params.Add(FString::Printf(TEXT("category=%s"), *Category));
    }
    if (!Search.IsEmpty())
    {
        Params.Add(FString::Printf(TEXT("search=%s"), *FGenericPlatformHttp::UrlEncode(Search)));
    }
    if (Params.Num() > 0)
    {
        URL += TEXT("?") + FString::Join(Params, TEXT("&"));
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(URL);
    Request->SetVerb(TEXT("GET"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnProductsResponse);
    Request->ProcessRequest();
    
    UE_LOG(LogTemp, Log, TEXT("GetProducts: %s"), *URL);
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnProductsResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    TArray<FMetaPulsProduct> Products;
    
    if (!bSuccess || !Response.IsValid() || Response->GetResponseCode() != 200)
    {
        UE_LOG(LogTemp, Error, TEXT("GetProducts failed"));
        OnProductsLoaded.Broadcast(false, 0, Products);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        OnProductsLoaded.Broadcast(false, 0, Products);
        return;
    }
    
    int32 NumberOfProducts = JsonObject->GetIntegerField(TEXT("NumberOfProducts"));
    
    const TArray<TSharedPtr<FJsonValue>>* ProductsArray;
    if (JsonObject->TryGetArrayField(TEXT("Products"), ProductsArray))
    {
        for (const TSharedPtr<FJsonValue>& ProductValue : *ProductsArray)
        {
            Products.Add(ParseProduct(ProductValue->AsObject()));
        }
    }
    
    UE_LOG(LogTemp, Log, TEXT("Loaded %d products"), Products.Num());
    OnProductsLoaded.Broadcast(true, NumberOfProducts, Products);
}

// Helper: Parse Product
FMetaPulsProduct UMetaPulsAPI::ParseProduct(const TSharedPtr<FJsonObject>& JsonObject)
{
    FMetaPulsProduct Product;
    
    Product.Id = JsonObject->GetIntegerField(TEXT("Id"));
    Product.Name = JsonObject->GetStringField(TEXT("Name"));
    Product.Price = JsonObject->GetNumberField(TEXT("Price"));
    Product.Currency = JsonObject->GetStringField(TEXT("Currency"));
    Product.PriceFormatted = JsonObject->GetStringField(TEXT("PriceFormatted"));
    
    // Files
    Product.Image = JsonObject->GetStringField(TEXT("Image"));
    Product.ImageUrl = JsonObject->GetStringField(TEXT("ImageUrl"));
    Product.Glb = JsonObject->GetStringField(TEXT("Glb"));
    Product.GlbUrl = JsonObject->GetStringField(TEXT("GlbUrl"));
    Product.Ply = JsonObject->GetStringField(TEXT("Ply"));
    Product.PlyUrl = JsonObject->GetStringField(TEXT("PlyUrl"));
    
    // Transform (separate X/Y/Z fields)
    Product.Location = FVector(
        JsonObject->GetNumberField(TEXT("LocationX")),
        JsonObject->GetNumberField(TEXT("LocationY")),
        JsonObject->GetNumberField(TEXT("LocationZ"))
    );
    
    Product.Rotation = FRotator(
        JsonObject->GetNumberField(TEXT("RotationX")),  // Pitch
        JsonObject->GetNumberField(TEXT("RotationZ")),  // Yaw
        JsonObject->GetNumberField(TEXT("RotationY"))   // Roll
    );
    
    Product.Scale = FVector(
        JsonObject->GetNumberField(TEXT("ScaleX")),
        JsonObject->GetNumberField(TEXT("ScaleY")),
        JsonObject->GetNumberField(TEXT("ScaleZ"))
    );
    
    return Product;
}
```

---

## 4.2 Get Single Product

🟢 **GET** `/api/tryon/products/{id}`  
**Auth Required:** No (Public)

### cURL

```bash
curl "https://meta-puls.com/api/tryon/products/2" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "Id": 2,
    "Name": "BAG-104",
    "Description": "Fashion product BAG-104",
    "Price": 45,
    "Currency": "EUR",
    "PriceFormatted": "€45.00",
    "Category": "dress",
    "Image": "BAG-104.jpg",
    "ImageUrl": "https://meta-puls.com/storage/shop-products/BAG-104.jpg",
    "Glb": "BAG-104.glb",
    "GlbUrl": "https://meta-puls.com/storage/shop-products/BAG-104.glb",
    "Ply": "BAG-104.ply",
    "PlyUrl": "https://meta-puls.com/storage/shop-products/BAG-104.ply",
    "LocationX": 0,
    "LocationY": 0,
    "LocationZ": 0,
    "RotationX": -90,
    "RotationY": 0,
    "RotationZ": 90,
    "ScaleX": 0.25,
    "ScaleY": 0.25,
    "ScaleZ": 0.25
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::GetProduct(int32 ProductId)
{
    FString URL = FString::Printf(TEXT("https://meta-puls.com/api/tryon/products/%d"), ProductId);
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(URL);
    Request->SetVerb(TEXT("GET"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnSingleProductResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnSingleProductResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid() || Response->GetResponseCode() != 200)
    {
        OnProductLoaded.Broadcast(false, FMetaPulsProduct());
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        OnProductLoaded.Broadcast(false, FMetaPulsProduct());
        return;
    }
    
    FMetaPulsProduct Product = ParseProduct(JsonObject);
    
    // Extra fields for single product
    Product.Description = JsonObject->GetStringField(TEXT("Description"));
    Product.Category = JsonObject->GetStringField(TEXT("Category"));
    
    UE_LOG(LogTemp, Log, TEXT("Product loaded: %s - %s"), *Product.Name, *Product.PriceFormatted);
    OnProductLoaded.Broadcast(true, Product);
}
```

---

# 5. TryOn List (Favorites)

## 5.1 Get TryOn List

🟢 **GET** `/api/tryon/list`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl "https://meta-puls.com/api/tryon/list" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "NumberOfItems": 2,
    "Items": [
        {
            "ListId": 6,
            "ProductId": 3,
            "AddedAt": "2025-12-31T09:07:53+00:00",
            "Product": {
                "Id": 3,
                "Name": "G-21",
                "Price": 32,
                "Currency": "EUR",
                "PriceFormatted": "€32.00",
                "Image": "G-21.jpg",
                "ImageUrl": "https://meta-puls.com/storage/shop-products/G-21.jpg",
                "Glb": "G-21.glb",
                "GlbUrl": "https://meta-puls.com/storage/shop-products/G-21.glb",
                "Ply": "G-21.ply",
                "PlyUrl": "https://meta-puls.com/storage/shop-products/G-21.ply",
                "LocationX": 0,
                "LocationY": 0,
                "LocationZ": 0,
                "RotationX": 0,
                "RotationY": 0,
                "RotationZ": 0,
                "ScaleX": 1,
                "ScaleY": 1,
                "ScaleZ": 1
            }
        },
        {
            "ListId": 5,
            "ProductId": 2,
            "AddedAt": "2025-12-31T09:07:52+00:00",
            "Product": {
                "Id": 2,
                "Name": "BAG-104",
                "Price": 45,
                "Currency": "EUR",
                "PriceFormatted": "€45.00",
                "Image": "BAG-104.jpg",
                "ImageUrl": "https://meta-puls.com/storage/shop-products/BAG-104.jpg",
                "Glb": "BAG-104.glb",
                "GlbUrl": "https://meta-puls.com/storage/shop-products/BAG-104.glb",
                "Ply": "BAG-104.ply",
                "PlyUrl": "https://meta-puls.com/storage/shop-products/BAG-104.ply",
                "LocationX": 0,
                "LocationY": 0,
                "LocationZ": 0,
                "RotationX": -90,
                "RotationY": 0,
                "RotationZ": 90,
                "ScaleX": 0.25,
                "ScaleY": 0.25,
                "ScaleZ": 0.25
            }
        }
    ]
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::GetTryOnList()
{
    if (AuthToken.IsEmpty())
    {
        OnTryOnListLoaded.Broadcast(false, 0, TArray<FMetaPulsTryOnItem>());
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/tryon/list"));
    Request->SetVerb(TEXT("GET"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnTryOnListResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnTryOnListResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    TArray<FMetaPulsTryOnItem> Items;
    
    if (!bSuccess || !Response.IsValid() || Response->GetResponseCode() != 200)
    {
        OnTryOnListLoaded.Broadcast(false, 0, Items);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        OnTryOnListLoaded.Broadcast(false, 0, Items);
        return;
    }
    
    int32 NumberOfItems = JsonObject->GetIntegerField(TEXT("NumberOfItems"));
    
    const TArray<TSharedPtr<FJsonValue>>* ItemsArray;
    if (JsonObject->TryGetArrayField(TEXT("Items"), ItemsArray))
    {
        for (const TSharedPtr<FJsonValue>& ItemValue : *ItemsArray)
        {
            const TSharedPtr<FJsonObject>& ItemObj = ItemValue->AsObject();
            
            FMetaPulsTryOnItem Item;
            Item.ListId = ItemObj->GetIntegerField(TEXT("ListId"));
            Item.ProductId = ItemObj->GetIntegerField(TEXT("ProductId"));
            Item.AddedAt = ItemObj->GetStringField(TEXT("AddedAt"));
            
            const TSharedPtr<FJsonObject>* ProductObj;
            if (ItemObj->TryGetObjectField(TEXT("Product"), ProductObj))
            {
                Item.Product = ParseProduct(*ProductObj);
            }
            
            Items.Add(Item);
        }
    }
    
    UE_LOG(LogTemp, Log, TEXT("TryOn list loaded: %d items"), Items.Num());
    OnTryOnListLoaded.Broadcast(true, NumberOfItems, Items);
}
```

---

## 5.2 Add to TryOn List

🔵 **POST** `/api/tryon/list`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X POST "https://meta-puls.com/api/tryon/list" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"product_id": 2}'
```

### Request Body

```json
{
    "product_id": 2
}
```

### Response (201 Created)

```json
{
    "success": true,
    "message": "Product added to TryOn list",
    "item": {
        "id": 5,
        "product_id": 2
    }
}
```

### Error Response (400)

```json
{
    "success": false,
    "message": "Product already in TryOn list"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::AddToTryOnList(int32 ProductId)
{
    if (AuthToken.IsEmpty())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Not authenticated"));
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/tryon/list"));
    Request->SetVerb(TEXT("POST"));
    Request->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    // JSON body
    TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
    JsonObject->SetNumberField(TEXT("product_id"), ProductId);
    
    FString RequestBody;
    TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&RequestBody);
    FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);
    Request->SetContentAsString(RequestBody);
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnTryOnAddResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnTryOnAddResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Connection failed"));
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        
        UE_LOG(LogTemp, Log, TEXT("AddToTryOnList: %s"), *Message);
        OnTryOnListUpdated.Broadcast(bSuccessField, Message);
    }
    else
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Invalid response"));
    }
}
```

---

## 5.3 Remove from TryOn List

🔴 **DELETE** `/api/tryon/list/{product_id}`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X DELETE "https://meta-puls.com/api/tryon/list/2" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "Product removed from TryOn list"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::RemoveFromTryOnList(int32 ProductId)
{
    if (AuthToken.IsEmpty())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Not authenticated"));
        return;
    }
    
    FString URL = FString::Printf(TEXT("https://meta-puls.com/api/tryon/list/%d"), ProductId);
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(URL);
    Request->SetVerb(TEXT("DELETE"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnTryOnRemoveResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnTryOnRemoveResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Connection failed"));
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        
        UE_LOG(LogTemp, Log, TEXT("RemoveFromTryOnList: %s"), *Message);
        OnTryOnListUpdated.Broadcast(bSuccessField, Message);
    }
}
```

---

## 5.4 Clear TryOn List

🔴 **DELETE** `/api/tryon/list`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X DELETE "https://meta-puls.com/api/tryon/list" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "TryOn list cleared"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::ClearTryOnList()
{
    if (AuthToken.IsEmpty())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Not authenticated"));
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/tryon/list"));
    Request->SetVerb(TEXT("DELETE"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnTryOnClearResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnTryOnClearResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnTryOnListUpdated.Broadcast(false, TEXT("Connection failed"));
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        
        UE_LOG(LogTemp, Log, TEXT("ClearTryOnList: %s"), *Message);
        OnTryOnListUpdated.Broadcast(bSuccessField, Message);
    }
}
```

---

# 6. Shopping Cart

## 6.1 Get Cart

🟢 **GET** `/api/cart`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl "https://meta-puls.com/api/cart" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "cart": {
        "id": 1,
        "items_count": 2,
        "total_quantity": 6,
        "total": 250,
        "total_formatted": "€250.00",
        "currency": "EUR"
    },
    "items": [
        {
            "id": 5,
            "product_id": 2,
            "quantity": 5,
            "product": {
                "id": 2,
                "name": "BAG-104",
                "sku": "BAG-104",
                "price": "45.00",
                "price_formatted": "€45.00",
                "image": "https://meta-puls.com/storage/shop-products/BAG-104.jpg",
                "glb_file": "https://meta-puls.com/storage/shop-products/BAG-104.glb"
            },
            "subtotal": 225,
            "subtotal_formatted": "€225.00"
        },
        {
            "id": 6,
            "product_id": 4,
            "quantity": 1,
            "product": {
                "id": 4,
                "name": "GR-100",
                "sku": "GR-100",
                "price": "25.00",
                "price_formatted": "€25.00",
                "image": "https://meta-puls.com/storage/shop-products/GR-100.jpg",
                "glb_file": "https://meta-puls.com/storage/shop-products/GR-100.glb"
            },
            "subtotal": 25,
            "subtotal_formatted": "€25.00"
        }
    ]
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::GetCart()
{
    if (AuthToken.IsEmpty())
    {
        OnCartLoaded.Broadcast(false, FMetaPulsCart());
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/cart"));
    Request->SetVerb(TEXT("GET"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnCartResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnCartResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    FMetaPulsCart Cart;
    
    if (!bSuccess || !Response.IsValid() || Response->GetResponseCode() != 200)
    {
        OnCartLoaded.Broadcast(false, Cart);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (!FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        OnCartLoaded.Broadcast(false, Cart);
        return;
    }
    
    // Parse cart info
    const TSharedPtr<FJsonObject>* CartObj;
    if (JsonObject->TryGetObjectField(TEXT("cart"), CartObj))
    {
        Cart.Id = (*CartObj)->GetIntegerField(TEXT("id"));
        Cart.ItemsCount = (*CartObj)->GetIntegerField(TEXT("items_count"));
        Cart.TotalQuantity = (*CartObj)->GetIntegerField(TEXT("total_quantity"));
        Cart.Total = (*CartObj)->GetNumberField(TEXT("total"));
        Cart.TotalFormatted = (*CartObj)->GetStringField(TEXT("total_formatted"));
        Cart.Currency = (*CartObj)->GetStringField(TEXT("currency"));
    }
    
    // Parse items
    const TArray<TSharedPtr<FJsonValue>>* ItemsArray;
    if (JsonObject->TryGetArrayField(TEXT("items"), ItemsArray))
    {
        for (const TSharedPtr<FJsonValue>& ItemValue : *ItemsArray)
        {
            const TSharedPtr<FJsonObject>& ItemObj = ItemValue->AsObject();
            
            FMetaPulsCartItem Item;
            Item.Id = ItemObj->GetIntegerField(TEXT("id"));
            Item.ProductId = ItemObj->GetIntegerField(TEXT("product_id"));
            Item.Quantity = ItemObj->GetIntegerField(TEXT("quantity"));
            Item.Subtotal = ItemObj->GetNumberField(TEXT("subtotal"));
            Item.SubtotalFormatted = ItemObj->GetStringField(TEXT("subtotal_formatted"));
            
            const TSharedPtr<FJsonObject>* ProductObj;
            if (ItemObj->TryGetObjectField(TEXT("product"), ProductObj))
            {
                Item.ProductName = (*ProductObj)->GetStringField(TEXT("name"));
                Item.ProductSku = (*ProductObj)->GetStringField(TEXT("sku"));
                Item.ProductPrice = FCString::Atof(*(*ProductObj)->GetStringField(TEXT("price")));
                Item.ProductImage = (*ProductObj)->GetStringField(TEXT("image"));
            }
            
            Cart.Items.Add(Item);
        }
    }
    
    UE_LOG(LogTemp, Log, TEXT("Cart loaded: %d items, Total: %s"), Cart.ItemsCount, *Cart.TotalFormatted);
    OnCartLoaded.Broadcast(true, Cart);
}
```

---

## 6.2 Add to Cart

🔵 **POST** `/api/cart`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X POST "https://meta-puls.com/api/cart" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"product_id": 2, "quantity": 2}'
```

### Request Body

```json
{
    "product_id": 2,
    "quantity": 2
}
```

### Response (201 Created)

```json
{
    "success": true,
    "message": "Product added to cart",
    "cart_item": {
        "id": 5,
        "product_id": 2,
        "quantity": 2
    },
    "cart_total": 90,
    "cart_total_formatted": "€90.00"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::AddToCart(int32 ProductId, int32 Quantity)
{
    if (AuthToken.IsEmpty())
    {
        OnCartUpdated.Broadcast(false, TEXT("Not authenticated"), 0);
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/cart"));
    Request->SetVerb(TEXT("POST"));
    Request->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    // JSON body
    TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
    JsonObject->SetNumberField(TEXT("product_id"), ProductId);
    JsonObject->SetNumberField(TEXT("quantity"), Quantity);
    
    FString RequestBody;
    TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&RequestBody);
    FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);
    Request->SetContentAsString(RequestBody);
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnCartAddResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnCartAddResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnCartUpdated.Broadcast(false, TEXT("Connection failed"), 0);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        float CartTotal = JsonObject->GetNumberField(TEXT("cart_total"));
        
        UE_LOG(LogTemp, Log, TEXT("AddToCart: %s, Total: %.2f"), *Message, CartTotal);
        OnCartUpdated.Broadcast(bSuccessField, Message, CartTotal);
    }
}
```

---

## 6.3 Update Cart Item

🟡 **PUT** `/api/cart/{product_id}`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X PUT "https://meta-puls.com/api/cart/2" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"quantity": 5}'
```

### Request Body

```json
{
    "quantity": 5
}
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "Cart updated",
    "cart_item": {
        "id": 5,
        "product_id": 2,
        "quantity": 5
    },
    "cart_total": 250,
    "cart_total_formatted": "€250.00"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::UpdateCartItem(int32 ProductId, int32 Quantity)
{
    if (AuthToken.IsEmpty())
    {
        OnCartUpdated.Broadcast(false, TEXT("Not authenticated"), 0);
        return;
    }
    
    FString URL = FString::Printf(TEXT("https://meta-puls.com/api/cart/%d"), ProductId);
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(URL);
    Request->SetVerb(TEXT("PUT"));
    Request->SetHeader(TEXT("Content-Type"), TEXT("application/json"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    // JSON body
    TSharedPtr<FJsonObject> JsonObject = MakeShareable(new FJsonObject);
    JsonObject->SetNumberField(TEXT("quantity"), Quantity);
    
    FString RequestBody;
    TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&RequestBody);
    FJsonSerializer::Serialize(JsonObject.ToSharedRef(), Writer);
    Request->SetContentAsString(RequestBody);
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnCartUpdateResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnCartUpdateResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnCartUpdated.Broadcast(false, TEXT("Connection failed"), 0);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        float CartTotal = JsonObject->GetNumberField(TEXT("cart_total"));
        
        UE_LOG(LogTemp, Log, TEXT("UpdateCartItem: %s, Total: %.2f"), *Message, CartTotal);
        OnCartUpdated.Broadcast(bSuccessField, Message, CartTotal);
    }
}
```

---

## 6.4 Remove from Cart

🔴 **DELETE** `/api/cart/{product_id}`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X DELETE "https://meta-puls.com/api/cart/2" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "Product removed from cart",
    "cart_total": 25,
    "cart_total_formatted": "€25.00"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::RemoveFromCart(int32 ProductId)
{
    if (AuthToken.IsEmpty())
    {
        OnCartUpdated.Broadcast(false, TEXT("Not authenticated"), 0);
        return;
    }
    
    FString URL = FString::Printf(TEXT("https://meta-puls.com/api/cart/%d"), ProductId);
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(URL);
    Request->SetVerb(TEXT("DELETE"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnCartRemoveResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnCartRemoveResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnCartUpdated.Broadcast(false, TEXT("Connection failed"), 0);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        float CartTotal = JsonObject->GetNumberField(TEXT("cart_total"));
        
        UE_LOG(LogTemp, Log, TEXT("RemoveFromCart: %s, Total: %.2f"), *Message, CartTotal);
        OnCartUpdated.Broadcast(bSuccessField, Message, CartTotal);
    }
}
```

---

## 6.5 Clear Cart

🔴 **DELETE** `/api/cart`  
**Auth Required:** ✅ Bearer Token

### cURL

```bash
curl -X DELETE "https://meta-puls.com/api/cart" \
  -H "Authorization: Bearer 7|98zPWYdMO7oVFg4vjYIJszyyPxBztxGdXjWC8DL182a76606" \
  -H "Accept: application/json"
```

### Response (200 OK)

```json
{
    "success": true,
    "message": "Cart cleared"
}
```

### Unreal C++ - Send Request

```cpp
void UMetaPulsAPI::ClearCart()
{
    if (AuthToken.IsEmpty())
    {
        OnCartUpdated.Broadcast(false, TEXT("Not authenticated"), 0);
        return;
    }
    
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL(TEXT("https://meta-puls.com/api/cart"));
    Request->SetVerb(TEXT("DELETE"));
    Request->SetHeader(TEXT("Accept"), TEXT("application/json"));
    Request->SetHeader(TEXT("Authorization"), FString::Printf(TEXT("Bearer %s"), *AuthToken));
    
    Request->OnProcessRequestComplete().BindUObject(this, &UMetaPulsAPI::OnCartClearResponse);
    Request->ProcessRequest();
}
```

### Unreal C++ - Handle Response

```cpp
void UMetaPulsAPI::OnCartClearResponse(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bSuccess)
{
    if (!bSuccess || !Response.IsValid())
    {
        OnCartUpdated.Broadcast(false, TEXT("Connection failed"), 0);
        return;
    }
    
    FString ResponseString = Response->GetContentAsString();
    TSharedPtr<FJsonObject> JsonObject;
    TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);
    
    if (FJsonSerializer::Deserialize(Reader, JsonObject))
    {
        bool bSuccessField = JsonObject->GetBoolField(TEXT("success"));
        FString Message = JsonObject->GetStringField(TEXT("message"));
        
        UE_LOG(LogTemp, Log, TEXT("ClearCart: %s"), *Message);
        OnCartUpdated.Broadcast(bSuccessField, Message, 0);
    }
}
```

---

# 7. Web Integration

## 7.1 Open TryOn Page with Token

You can automatically log into web pages using the API token.

### URL Format

```
https://meta-puls.com/en/tryon?token=YOUR_TOKEN_HERE
https://meta-puls.com/de/tryon?token=YOUR_TOKEN_HERE
```

### Unreal C++ - Open Browser

```cpp
void UMetaPulsAPI::OpenTryOnWebPage(const FString& Language)
{
    if (AuthToken.IsEmpty())
    {
        UE_LOG(LogTemp, Warning, TEXT("Cannot open TryOn page: Not logged in"));
        return;
    }
    
    // Default to English
    FString Lang = Language.IsEmpty() ? TEXT("en") : Language;
    
    FString URL = FString::Printf(
        TEXT("https://meta-puls.com/%s/tryon?token=%s"), 
        *Lang, 
        *AuthToken
    );
    
    UE_LOG(LogTemp, Log, TEXT("Opening TryOn page: %s"), *URL);
    
    // Open in default browser
    FPlatformProcess::LaunchURL(*URL, nullptr, nullptr);
}
```

### Usage in Blueprint/C++

```cpp
// After successful login
void AMyGameMode::OnLoginSuccess()
{
    UMetaPulsAPI* API = GetGameInstance()->GetSubsystem<UMetaPulsAPI>();
    
    // Open English TryOn page
    API->OpenTryOnWebPage(TEXT("en"));
    
    // Or German
    // API->OpenTryOnWebPage(TEXT("de"));
}
```

---

# 📊 API Endpoints Summary

| # | Method | Endpoint | Auth | Description |
|---|--------|----------|:----:|-------------|
| 1 | 🔵 POST | `/api/login` | ❌ | Login, get token + houses |
| 2 | 🔵 POST | `/api/logout` | ✅ | Logout, invalidate token |
| 3 | 🟢 GET | `/api/profile` | ✅ | Get profile + houses |
| 4 | 🟢 GET | `/api/tryon/products` | ❌ | List all products |
| 5 | 🟢 GET | `/api/tryon/products/{id}` | ❌ | Get single product |
| 6 | 🟢 GET | `/api/tryon/list` | ✅ | Get favorites |
| 7 | 🔵 POST | `/api/tryon/list` | ✅ | Add to favorites |
| 8 | 🔴 DELETE | `/api/tryon/list/{id}` | ✅ | Remove from favorites |
| 9 | 🔴 DELETE | `/api/tryon/list` | ✅ | Clear favorites |
| 10 | 🟢 GET | `/api/cart` | ✅ | Get cart |
| 11 | 🔵 POST | `/api/cart` | ✅ | Add to cart |
| 12 | 🟡 PUT | `/api/cart/{id}` | ✅ | Update quantity |
| 13 | 🔴 DELETE | `/api/cart/{id}` | ✅ | Remove from cart |
| 14 | 🔴 DELETE | `/api/cart` | ✅ | Clear cart |

---

# ✅ Checklist

| Endpoint | cURL | Request | Response | UE Send | UE Parse |
|----------|:----:|:-------:|:--------:|:-------:|:--------:|
| Login | ✅ | ✅ | ✅ | ✅ | ✅ |
| Logout | ✅ | - | ✅ | ✅ | ✅ |
| Profile | ✅ | - | ✅ | ✅ | ✅ |
| Products List | ✅ | - | ✅ | ✅ | ✅ |
| Product Single | ✅ | - | ✅ | ✅ | ✅ |
| TryOn Get | ✅ | - | ✅ | ✅ | ✅ |
| TryOn Add | ✅ | ✅ | ✅ | ✅ | ✅ |
| TryOn Remove | ✅ | - | ✅ | ✅ | ✅ |
| TryOn Clear | ✅ | - | ✅ | ✅ | ✅ |
| Cart Get | ✅ | - | ✅ | ✅ | ✅ |
| Cart Add | ✅ | ✅ | ✅ | ✅ | ✅ |
| Cart Update | ✅ | ✅ | ✅ | ✅ | ✅ |
| Cart Remove | ✅ | - | ✅ | ✅ | ✅ |
| Cart Clear | ✅ | - | ✅ | ✅ | ✅ |
| Web TryOn | ✅ | - | - | ✅ | - |

---

*© 2025 Meta-Puls GmbH. All rights reserved.*
