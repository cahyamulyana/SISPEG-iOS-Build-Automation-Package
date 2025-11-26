#!/bin/bash

# SISPEG iOS Production Build Script
echo "🚀 Building SISPEG iOS Production App..."

# Set version
VERSION="2.1.0"
BUILD_DATE=$(date +%Y%m%d_%H%M%S)

echo "📦 Version: $VERSION"
echo "📅 Build Date: $BUILD_DATE"

# Check if running on macOS
if [[ "$OSTYPE" != "darwin"* ]]; then
    echo "❌ Error: iOS builds require macOS with Xcode"
    echo "💡 Use one of these options:"
    echo "   1. Run on Mac computer with Xcode"
    echo "   2. Use Codemagic CI/CD (recommended)"
    echo "   3. Use GitHub Actions with macOS runner"
    echo "   4. Use App Center"
    exit 1
fi

# Check if Xcode is installed
if ! command -v xcodebuild &> /dev/null; then
    echo "❌ Xcode not found. Please install Xcode from Mac App Store"
    exit 1
fi

# Navigate to cordova project
cd cordova_project

# Clean previous builds
echo "🧹 Cleaning previous builds..."
rm -rf platforms/ios/build
rm -rf platforms/ios/www

# Prepare iOS platform
echo "📱 Preparing iOS platform..."
cordova platform remove ios --nosave 2>/dev/null
cordova platform add ios@7.1.1 --nosave

# Build production app
echo "🔨 Building iOS production app..."
cordova build ios --release --device

# Check if build was successful
if [ -d "platforms/ios/build/device" ]; then
    echo "✅ iOS build successful!"

    # Create build directory
    BUILD_DIR="../../ios_build_$BUILD_DATE"
    mkdir -p "$BUILD_DIR"

    # Copy build to mobile directory
    cp -r platforms/ios/build/device/* "$BUILD_DIR/"

    echo "📂 Build output copied to: $BUILD_DIR/"
    echo ""
    echo "📋 Next steps for App Store submission:"
    echo "1. Open Xcode project: $BUILD_DIR/SISPEG.xcodeproj"
    echo "2. Configure signing & capabilities in Xcode"
    echo "3. Archive the app in Xcode (Product > Archive)"
    echo "4. Upload to App Store Connect via Xcode Organizer"
    echo ""
    echo "🎯 App Store ID: Configure in config.xml and index.html"
    echo "🔗 Update URL: https://sispeg.sistemkeuangan.com/mobile/ios-update-check.json"
    echo ""
    echo "📱 For TestFlight distribution:"
    echo "1. After archiving, select 'Distribute App'"
    echo "2. Choose 'TestFlight & App Store'"
    echo "3. Follow the upload process"
    echo ""
    echo "🎉 iOS production build completed!"
else
    echo "❌ iOS build failed!"
    echo "💡 Check the error messages above and ensure:"
    echo "   - Xcode is properly installed"
    echo "   - Command Line Tools are installed (xcode-select --install)"
    echo "   - You have accepted Xcode license (sudo xcodebuild -license accept)"
    echo "   - CocoaPods is installed if needed (sudo gem install cocoapods)"
    exit 1
fi
