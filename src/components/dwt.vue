<template>
  <div id="DWTcontainer" class="container">
    <button id="btnScan" @click="acquireImage()" style="margin: 20px">Scan</button>
    <button id="btnLoad" @click="loadImage()" style="margin-right: 20px">Load</button>
    <button id="btnReadBarcode" @click="readBarcode()">Read Barcode</button>
    <div id="dwtcontrolContainer"></div>
    <div
      id="divNoteMessage"
      ondblclick="this.innerHTML='';"
      style="
        margin: 20px auto;
        width: 550px;
        height: 250px;
        overflow: auto;
        background-color: #e7f2fd;
        border: solid 1px black;
        text-align: left;
      "
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
      dbrObject: false,
    };
  },
  created() {
    /**
     * ResourcesPath must be set in order to use the library!
     */
    Dynamsoft.DWT.ResourcesPath = "dwt-resources";
    Dynamsoft.DWT.AutoLoad = false;
	
    /** v17.1 LICENSE ALERT - README
     * The library requires a license to work, the APIs organizationID and handshakeCode specify how to acquire a license.
     * If nothing is specified, a 7-day (public) trial license will be used by default which is the case in this sample.
     * Note that network connection is required for this license to work.
     */

    /* When using your own license, please uncomment the following lines and fill in your own information. */
    /* To get a free trial, please visit https://www.dynamsoft.com/customer/license/trialLicense?product=dwt. */

    // Dynamsoft.DWT.organizationID = "YOUR-ORGANIZATION-ID";
    // Dynamsoft.DWT.handshakeCode = "A-SPECIFIC-HANDSHAKECODE";
    // Dynamsoft.DWT.sessionPassword = "PASSWORD-TO-PROTECT-YOUR-LICENSE"; // Important field to protect your license.
    // Dynamsoft.DWT.licenseServer = ["YOUR-OWN-MAIN-LTS", "YOUR-OWN-STANDBY-LTS"]; // Ignore this line if you are using Dynamsoft-hosting LTS

    /* The API "ProductKey" is an alternative way to license the library, the major difference is that it does not require a network. Contact support@dynamsoft.com for more information. */

    // Dynamsoft.DWT.ProductKey = "YOUR-PRODUCT-KEY";

    /** LICENSE ALERT - THE END */
	
    Dynamsoft.DWT.Containers = [
      { ContainerId: "dwtcontrolContainer", Width: "550px", Height: "513px" },
    ];
    Dynamsoft.DWT.RegisterEvent("OnWebTwainReady", () => {
      this.Dynamsoft_OnReady();
    });
  },
  mounted() {
    Dynamsoft.DWT.Load();
  },
  methods: {
    Dynamsoft_OnReady() {
      this.DWObject = Dynamsoft.DWT.GetWebTwain("dwtcontrolContainer");
    },

    // Acquire Image
    acquireImage() {
        var _this = this;
      _this.DWObject.SelectSource(function () {
		_this.DWObject.IfDisableSourceAfterAcquire = true;
		_this.DWObject.AcquireImage();
     }, function () {
		alert('SelectSource failed!');
     });
    },

    // Load Images
    loadImage() {
      this.DWObject.IfShowFileDialog = true;
      this.DWObject.Addon.PDF.SetResolution(300);
      this.DWObject.Addon.PDF.SetConvertMode(Dynamsoft.DWT.EnumDWT_ConvertMode.CM_RENDERALL);
      this.DWObject.LoadImageEx("", 5);
    },

    // Read Barcode
    async readBarcode() {
      if (this.DWObject) {
        if (this.DWObject.HowManyImagesInBuffer == 0) {
          alert("Please scan or load an image first.");
          return;
        }
        var settings = await this.DWObject.Addon.BarcodeReader.getRuntimeSettings();
        /**
         * Setting up the barcode reader
         */
        settings.barcodeFormatIds = Dynamsoft.DBR.EnumBarcodeFormat.BF_ALL;
        settings.deblurLevel = 9;
        settings.expectedBarcodesCount = 512;
        settings.scaleDownThreshold = 48000;
        /** End of settings */
        await this.DWObject.Addon.BarcodeReader.updateRuntimeSettings(settings);
        var index = this.DWObject.CurrentImageIndexInBuffer;
        var objDiv = document.getElementById("divNoteMessage");
        this.DWObject.Addon.BarcodeReader.decode(index).then(
          function (results) {
            //This is the function called when barcode is read successfully
            //Retrieve barcode details
            var _now = new Date().toLocaleTimeString() + "<br />";
            objDiv.innerHTML += _now;
            if (results.length == 0) {
              objDiv.innerHTML += "Looking for a barcode of the specified type...<br />";
            } else {
              objDiv.innerHTML += "<p style='color: #fe8e14'>Barcode Found!!</p>";
              var barcodeText = [];
              for (var i = 0; i < results.length; i++) {
                var result = results[i];
                var Barcode_text = result.barcodeText;
                var format = result.barcodeFormatString;
                barcodeText.push("barcode[" + (i + 1) + "]: " + "<br />");
                barcodeText.push("<strong>" + Barcode_text + "</strong>");
                barcodeText.push("<br />format:" + format + "<br />");
                barcodeText.push("------------------------------<br />");
              }
              barcodeText.splice(0, 0, '<p style="padding:5px; margin:0;">');
              barcodeText.push("</p>");
              objDiv.innerHTML += barcodeText.join("");
            }
            objDiv.scrollTop = objDiv.scrollHeight;
          },
          function (ex) {
            //This is the function called when barcode reading fails
            objDiv.innerHTML = ex.message || ex;
          }
        );
      }
    },
  },
};
</script>

<style>
#dwtcontrolContainer > div {
  margin: 0 auto;
}
</style>
