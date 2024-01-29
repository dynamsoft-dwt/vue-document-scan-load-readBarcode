<template>
  <div id="DWTcontainer" class="container">
    <button id="btnScan" @click="acquireImage()" style="margin: 20px">Scan</button>
    <button id="btnLoad" @click="loadImage()" style="margin-right: 20px">Load</button>
    <button id="btnReadBarcode" @click="readBarcode()">Read Barcode</button>
    <div id="dwtcontrolContainer" style="width:550px;margin: 0 auto;"></div>
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
  name: "web-twain",
  data() {
    return {
      title: "Using Dynamic Web TWAIN in Vue Project",
      DWObject: false,
      dbrObject: false,
    };
  },
  created() {
    Dynamsoft.OnLicenseError = function (message, errorCode) {
      if(errorCode == -2808)
        message = '<div style="padding:0">Sorry. Your product key has expired. You can purchase a full license at the <a target="_blank" href="https://www.dynamsoft.com/store/dynamic-web-twain/#DynamicWebTWAIN">online store</a>.</div><div style="padding:0">Or, you can try requesting a new product key at <a target="_blank" href="https://www.dynamsoft.com/customer/license/trialLicense?product=dwt&utm_source=in-product">this page</a>.</div><div style="padding:0">If you need any help, please <a target="_blank" href="https://www.dynamsoft.com/company/contact/">contact us</a>.</div>';
        Dynamsoft.DWT.ShowMessage(message, {
        width: 680,
        headerStyle: 2
      });
    };
    /**
     * ResourcesPath & ProductKey must be set in order to use the library!
     */
    Dynamsoft.DWT.ResourcesPath = "/dwt-resources";
    Dynamsoft.DWT.AutoLoad = false;
    Dynamsoft.DWT.ProductKey = 'DLS2eyJvcmdhbml6YXRpb25JRCI6IjIwMDAwMSJ9';
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
      this.DWObject.SelectSourceAsync()
        .then(
          () => {return this.DWObject.AcquireImageAsync({
            IfDisableSourceAfterAcquire: true
          })}
        )
        .catch(
          (exp) =>{alert(exp.message)}
        );
    },

    // Load Images
    loadImage() {
      this.DWObject.IfShowFileDialog = true;
      this.DWObject.Addon.PDF.SetReaderOptions({
        convertMode: Dynamsoft.DWT.EnumDWT_ConvertMode.CM_RENDERALL,
        renderOptions: {
            resolution: 300
        }
      });
      this.DWObject.LoadImageEx("", 5);
    },

    // Read Barcode
    async readBarcode() {
      if (this.DWObject) {
        if (this.DWObject.HowManyImagesInBuffer == 0) {
          alert("Please scan or load an image first.");
          return;
        }
        const settings = await this.DWObject.Addon.BarcodeReader.getRuntimeSettings();
        /**
         * Setting up the barcode reader
         */
        settings.barcodeFormatIds = Dynamsoft.DBR.EnumBarcodeFormat.BF_ALL;
        settings.barcodeFormatIds_2 = Dynamsoft.DBR.EnumBarcodeFormat_2.BF2_DOTCODE + Dynamsoft.DBR.EnumBarcodeFormat_2.BF2_POSTALCODE;
        settings.deblurLevel = 9;
        settings.expectedBarcodesCount = 512;
        settings.scaleDownThreshold = 48000;
        /** End of settings */
        await this.DWObject.Addon.BarcodeReader.updateRuntimeSettings(settings);
        const index = this.DWObject.CurrentImageIndexInBuffer;
        const objDiv = document.getElementById("divNoteMessage");
        this.DWObject.Addon.BarcodeReader.decode(index).then(
          function (results) {
            //This is the function called when barcode is read successfully
            //Retrieve barcode details
            const now = new Date().toLocaleTimeString() + "<br />";
            objDiv.innerHTML += now;
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
