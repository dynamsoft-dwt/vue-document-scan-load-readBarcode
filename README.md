# vue-simple-document-scan

This project demonstrates how to build a Vue scanning project with [Dynamsoft Web TWAIN SDK](https://www.dynamsoft.com/web-twain/overview/).

## Usage
Environment: Node.js v22.14.0
1. Apply for a [30-day free trial license](https://www.dynamsoft.com/customer/license/trialLicense?product=dwt) of Dynamic Web TWAIN.

2. Update the license key in `src\components\dwt.vue` file:

   ```
   Dynamsoft.DWT.ProductKey = "LICENSE-KEY";
   ```

3. Install the dependencies:

   ```
   npm install --force
   ```

4. Run the application as follows:

   ```
   npm run serve
   ```


## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.