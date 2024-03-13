---
title: "[XCode] Building an XCFramework from a Static library"
excerpt: ".a Static Library 에서 XCFramework로 빌드"
description: "Building an XCFramework from a Static library"
modified: 2024-03-13
categories: "XCode"
tags: [XCode, XCFramework, StaticLibrary]

toc: false

header:
  teaser: /assets/images/teaser/ios-teaser.png
---

# 1. Archive Framework
## Copy the public headers into the framework

```shell
export FRAMEWORK_PATH="${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.framework"

export LIBRARY_DIST_VERSION=${FRAMEWORK_VERSION}

mkdir -p "${FRAMEWORK_PATH}/Versions/${LIBRARY_DIST_VERSION}/Headers"

/bin/ln -sfh "${LIBRARY_DIST_VERSION}" "${FRAMEWORK_PATH}/Versions/Current"
/bin/ln -sfh Versions/Current/Headers "${FRAMEWORK_PATH}/Headers"
/bin/ln -sfh "Versions/${PRODUCT_NAME}" \
"${FRAMEWORK_PATH}/${PRODUCT_NAME}"


/bin/cp -a "${TARGET_BUILD_DIR}/${PUBLIC_HEADERS_FOLDER_PATH}/" \
"${FRAMEWORK_PATH}/Versions/${LIBRARY_DIST_VERSION}/Headers"
```

## Make modulemap for Framework

```shell
mkdir -p "${FRAMEWORK_PATH}/Modules"
cat <<EOF > "${FRAMEWORK_PATH}/Modules/module.modulemap"
module ${PRODUCT_NAME} {
    header "Headers/${PRODUCT_NAME}.h"
    export *
}
EOF
```

## Copy the PrivacyInfo.xcprivacy file

```shell
cp ${SRCROOT}/PrivacyInfo.xcprivacy ${FRAMEWORK_PATH}/PrivacyInfo.xcprivacy
```

# 2. Build XCFramework
- SDK must be archived for device and for simulator before building the xcframework

## SDK Archive for device and for simulator

```shell
xcodebuild -target "${PRODUCT_NAME}" \
-configuration Release -arch arm64 only_active_arch=no defines_module=yes \
-sdk "iphoneos"

xcodebuild -target "${PRODUCT_NAME}" \
-configuration Release -arch x86_64 -arch arm64 only_active_arch=no defines_module=yes \
-sdk "iphonesimulator"
```

## Create XCFramework

```shell
FRAMEWORK_NAME=${PRODUCT_NAME}
BUILD_PATH="${SRCROOT}/build"
RELEASE_PATH="../xcframework"
IPHONEOS_PATH="${BUILD_PATH}/Device"
IPHONESIMULATOR_PATH="${BUILD_PATH}/Simulator"

if [ -d "${BUILD_PATH}" ]; then
rm -rf "${BUILD_PATH}"
fi

if ! [ -d "${RELEASE_PATH}" ]; then
mkdir -p "${RELEASE_PATH}"
fi

if ! [ -d "${IPHONEOS_PATH}" ]; then
mkdir -p "${IPHONEOS_PATH}"
fi

if ! [ -d "${IPHONESIMULATOR_PATH}" ]; then
mkdir -p "${IPHONESIMULATOR_PATH}"
fi

echo "Remove .xcframework file if exists from previous run"

if [ -d "${RELEASE_PATH}/${FRAMEWORK_NAME}.xcframework" ]; then
rm -rf "${RELEASE_PATH}/${FRAMEWORK_NAME}.xcframework"
fi

cp -R "${BUILD_PATH}/Release-iphoneos/${FRAMEWORK_NAME}.framework" "${IPHONEOS_PATH}/${FRAMEWORK_NAME}.framework"
cp -R "${BUILD_PATH}/Release-iphonesimulator/${FRAMEWORK_NAME}.framework" "${IPHONESIMULATOR_PATH}/${FRAMEWORK_NAME}.framework"

echo "LIPO Device"
lipo -create "${BUILD_PATH}/Release-iphoneos/lib${FRAMEWORK_NAME}.a" \
-output "${IPHONEOS_PATH}/${FRAMEWORK_NAME}.framework/Versions/${LIBRARY_DIST_VERSION}/${FRAMEWORK_NAME}"

echo "LIPO Simulator"
lipo -create "${BUILD_PATH}/Release-iphonesimulator/lib${FRAMEWORK_NAME}.a" \
-output "${IPHONESIMULATOR_PATH}/${FRAMEWORK_NAME}.framework/Versions/${LIBRARY_DIST_VERSION}/${FRAMEWORK_NAME}"

echo "Create xcframework"
xcodebuild -create-xcframework \
    -framework "${IPHONEOS_PATH}/${FRAMEWORK_NAME}.framework" \
    -framework "${IPHONESIMULATOR_PATH}/${FRAMEWORK_NAME}.framework" \
    -output "${RELEASE_PATH}/${FRAMEWORK_NAME}.xcframework"
```

## XCFramework Code Signing

```shell
codesign --timestamp -s "Apple Distribution: ..." ../xcframework/${PRODUCT_NAME}.xcframework
```
- [https://tigi44.github.io/ios/iOS-Code-Signing-XCFramework/](https://tigi44.github.io/ios/iOS-Code-Signing-XCFramework/){: target="_blank"}

# ex) Build FatFramework

```shell
FRAMEWORK_NAME=${PRODUCT_NAME}
BUILD_PATH="${SRCROOT}/build"
RELEASE_PATH="../Framework"

if [ -d "${BUILD_PATH}" ]; then
rm -rf "${BUILD_PATH}"
fi

xcodebuild -target "${FRAMEWORK_NAME}" \
-configuration Release -arch arm64 only_active_arch=no defines_module=yes \
-sdk "iphoneos"

xcodebuild -target "${FRAMEWORK_NAME}" \
-configuration Release -arch x86_64 only_active_arch=no defines_module=yes \
-sdk "iphonesimulator"

if ! [ -d "${RELEASE_PATH}" ]; then
mkdir -p "${RELEASE_PATH}"
fi

if [ -d "${RELEASE_PATH}/${FRAMEWORK_NAME}.framework" ]; then
rm -rf "${RELEASE_PATH}/${FRAMEWORK_NAME}.framework"
fi

cp -R "${BUILD_PATH}/Release-iphoneos/${FRAMEWORK_NAME}.framework" "${RELEASE_PATH}/${FRAMEWORK_NAME}.framework"

lipo -create "${BUILD_PATH}/Release-iphoneos/lib${FRAMEWORK_NAME}.a" "${BUILD_PATH}/Release-iphonesimulator/lib${FRAMEWORK_NAME}.a" \
-output "${RELEASE_PATH}/${FRAMEWORK_NAME}.framework/Versions/${LIBRARY_DIST_VERSION}/${FRAMEWORK_NAME}"
```
