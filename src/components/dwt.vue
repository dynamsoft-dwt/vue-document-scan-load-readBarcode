<template>
  <div id="DWTcontainer" class="container">
    <button id="btnScan" @click="acquireImage()" style="margin:20px;">Scan</button>
    <button id="btnLoad" @click="loadImage()" style="margin-right:20px;">Load</button>
    <button id="btnReadBarcode" @click="readBarcode()">Read Barcode</button>
    <div id="dwtcontrolContainer"></div>
    <div
      id="divNoteMessage"
      ondblclick="this.innerHTML='';"
      style="margin:20px auto;width:550px; height:250px; overflow: auto;background-color:#e7f2fd;border:solid 1px black;"
    ></div>
  </div>
</template>

<script>
import Dynamsoft from "dwt";

export default {
  name: "dwt",
  data() {
    return {
      title: "Using Dynamic Web TWAIN in Vue Project",
      DWObject: false,
      dbrObject: false
    };
  },
  created() {
    Dynamsoft.WebTwainEnv.AutoLoad = false;
    Dynamsoft.WebTwainEnv.ProductKey =
      "t0072dAAAAABaBr9lBs/fQuUDhzKLLueb8zc9jIWbJrTRLsJ+Wlr7dOYIGBgv0ZAaO5PR0QwesMx6nhsvp89nXXL6BUDqPiNbFRdw";
    Dynamsoft.WebTwainEnv.Containers = [
      { ContainerId: "dwtcontrolContainer", Width: "550px", Height: "513px" }
    ];
    Dynamsoft.WebTwainEnv.RegisterEvent("OnWebTwainReady", () => {
      this.Dynamsoft_OnReady();
    });
    dynamsoft.dbrEnv.productKey = Dynamsoft.WebTwainEnv.ProductKey;
    dynamsoft.dbrEnv.bAutoConnectService = false;
    /// set resourcesPath in runtime is ok

    var p = document.location.protocol;
    if (p !== "https:" && p !== "http:")
      dynamsoft.dbrEnv.resourcesPath = "https://tst.dynamsoft.com/libs/dbr/7.1";
    else
      dynamsoft.dbrEnv.resourcesPath = "http://tst.dynamsoft.com/libs/dbr/7.1";
  },
  mounted() {
    Dynamsoft.WebTwainEnv.Load();
  },
  methods: {
    Dynamsoft_OnReady() {
      this.DWObject = Dynamsoft.WebTwainEnv.GetWebTwain("dwtcontrolContainer");
      dynamsoft.dbrEnv.productKey = Dynamsoft.WebTwainEnv.ProductKey;
      dynamsoft.BarcodeReader.initServiceConnection().then(
        () => {
          this.dbrObject = new dynamsoft.BarcodeReader();
        },
        ex => {
          alert("Init failed: " + (ex.message || ex));
        }
      );
    },

    // Acquire Image
    acquireImage() {
      this.DWObject.IfDisableSourceAfterAcquire = true;
      this.DWObject.AcquireImage();
    },

    // Load Images
    loadImage() {
      this.DWObject.IfShowFileDialog = true;
      this.DWObject.Addon.PDF.SetResolution(300);
      this.DWObject.Addon.PDF.SetConvertMode(EnumDWT_ConvertMode.CM_RENDERALL);
      this.DWObject.LoadImageEx("", 5);
    },

    // Read Barcode
    readBarcode() {
      if (this.DWObject) {
        if (this.DWObject.HowManyImagesInBuffer == 0) {
          alert("Please scan or load an image first.");
          return;
        }
        var settings = this.dbrObject.getRuntimeSettings();
        /**
         * Setting up the barcode reader
         */
        settings.mBarcodeFormatIds = dynamsoft.BarcodeReader.EnumBarcodeFormat.All;
        settings.mAntiDamageLevel = 9;
        settings.mBarcodeInvertMode = 0;
        settings.mBinarizationBlockSize = 0;
        settings.mColourImageConvertMode = 0;
        settings.mDeblurLevel = 9;
        settings.mEnableFillBinaryVacancy = 1;
        settings.mExpectedBarcodesCount = 512;
        settings.mGrayEqualizationSensitivity = 0;
        settings.mLocalizationAlgorithmPriority = "";
        settings.mMaxAlgorithmThreadCount = 4;
        settings.mMaxBarcodesCount = 2147483647;
        settings.mMaxDimOfFullImageAsBarcodeZone = 262144;
        settings.mPDFRasterDPI = 300;
        settings.mRegionPredetectionMode = 1;
        settings.mScaleDownThreshold = 2147483647;
        settings.mTextFilterMode = 2;
        settings.mTextureDetectionSensitivity = 5;
        settings.mTimeout = 10000;
        /** End of settings */
        this.dbrObject.updateRuntimeSettings(settings);
        var index = this.DWObject.CurrentImageIndexInBuffer;
        var barcodeImage = this.DWObject.GetImagePartURL(index);
        var objDiv = document.getElementById("divNoteMessage");
        this.dbrObject.decode(barcodeImage).then(
          function(results) {
            //This is the function called when barcode is read successfully
            //Retrieve barcode details
            var _now = new Date().toLocaleTimeString() + "<br />";
            objDiv.innerHTML += _now;
            if (results.length == 0) {
              objDiv.innerHTML +=
                "Looking for a barcode of the specified type...<br />";
            } else {
              objDiv.innerHTML +=
                "<p style='color: #fe8e14'>Barcode Found!!</p>";
              var barcodeText = [];
              for (var i = 0; i < results.length; i++) {
                var result = results[i];
                var Barcode_text = result.BarcodeText;
                var loc = result.LocalizationResult;
                var x = loc.X1;
                var y = loc.Y1;
                var format = result.BarcodeFormatString;
                barcodeText.push("barcode[" + (i + 1) + "]: " + "<br />");
                barcodeText.push("<strong>" + Barcode_text + "</strong>");
                barcodeText.push("<br />format:" + format + "<br />");
                barcodeText.push("x:" + x + "y:" + y + "<br />");
                barcodeText.push("------------------------------<br />");
              }
              barcodeText.splice(0, 0, '<p style="padding:5px; margin:0;">');
              barcodeText.push("</p>");
              objDiv.innerHTML += barcodeText.join("");
            }
            objDiv.scrollTop = objDiv.scrollHeight;
          },
          function(ex) {
            //This is the function called when barcode reading fails
            objDiv.innerHTML = ex.message || ex;
          }
        );
      }
    }
  }
};
</script>

<style>
#dwtcontrolContainer > div {
  margin: 0 auto;
}
</style>
