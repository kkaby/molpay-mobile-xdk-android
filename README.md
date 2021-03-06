<!--
# license: Copyright © 2011-2016 MOLPay Sdn Bhd. All Rights Reserved. 
-->

# molpay-mobile-xdk-android

This is the complete and functional MOLPay Android payment module that is ready to be implemented into Android Studio through gradle from JCenter/Maven repository as a MOLPayXDK module. An example application project 
(MOLPayXdkExample) is provided for MOLPayXDK framework integration reference.

## Recommended configurations

    - Minimum Android SDK Version: 23 ++
    
    - Minimum Android API level: 16 ++
    
    - Minimum Android target version: Android 4.1

## MOLPay Android Caveats

    Credit card payment channel is not available in Android 4.1, 4.2, and 4.3. due to lack of latest security (TLS 1.2) support on these Android platforms natively.

## Installation

    Step 1 - Add compile 'com.molpay:molpay-mobile-xdk-android:<put latest release version here>' to dependencies in application build.gradle
    
    Step 2 - Add import com.molpay.molpayxdk.MOLPayActivity;
    
    Step 3 - Add the result callback function to get return results when the payment activity ended,
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data)
    {
        if (requestCode == MOLPayActivity.MOLPayXDK && resultCode == RESULT_OK){
            Log.d(MOLPayActivity.MOLPAY, "MOLPay result = "+data.getStringExtra(MOLPayActivity.MOLPayTransactionResult));
        }
    }
    
    =========================================
    Sample transaction result in JSON string:
    =========================================
    
    {"status_code":"11","amount":"1.01","chksum":"34a9ec11a5b79f31a15176ffbcac76cd","pInstruction":0,"msgType":"C6","paydate":1459240430,"order_id":"3q3rux7dj","err_desc":"","channel":"Credit","app_code":"439187","txn_ID":"6936766"}
    
    Parameter and meaning:
    
    "status_code" - "00" for Success, "11" for Failed, "22" for *Pending. 
    (*Pending status only applicable to cash channels only)
    "amount" - The transaction amount
    "paydate" - The transaction date
    "order_id" - The transaction order id
    "channel" - The transaction channel description
    "txn_ID" - The transaction id generated by MOLPay
    
    * Notes: You may ignore other parameters and values not stated above
    
    =====================================
    * Sample error result in JSON string:
    =====================================
    
    {"Error":"Communication Error"}
    
    Parameter and meaning:
    
    "Communication Error" - Error starting a payment process due to several possible reasons, please contact MOLPay support should the error persists.
    1) Internet not available
    2) API credentials (username, password, merchant id, verify key)
    3) MOLPay server offline.

## Prepare the Payment detail object

    HashMap<String, Object> paymentDetails = new HashMap<>();
        
        // Mandatory String. A value not less than '1.00'
        paymentDetails.put(MOLPayActivity.mp_amount, "1.10"); 
        
        // Mandatory String. Values obtained from MOLPay
        paymentDetails.put(MOLPayActivity.mp_username, "username");
        paymentDetails.put(MOLPayActivity.mp_password, "password");
        paymentDetails.put(MOLPayActivity.mp_merchant_ID, "merchantid");
        paymentDetails.put(MOLPayActivity.mp_app_name, "appname");
        paymentDetails.put(MOLPayActivity.mp_verification_key, "vkey123");
    
        // Mandatory String. Payment values
        paymentDetails.put(MOLPayActivity.mp_order_ID, "orderid123");
        paymentDetails.put(MOLPayActivity.mp_currency, "MYR");
        paymentDetails.put(MOLPayActivity.mp_country, "MY");
        
        // Optional String.
        paymentDetails.put(MOLPayActivity.mp_channel, "multi"); // Use 'multi' for all available channels option. For individual channel seletion, please refer to "Channel Parameter" in "Channel Lists" in the MOLPay API Spec for Merchant pdf. 
        paymentDetails.put(MOLPayActivity.mp_bill_description, "billdesc");
        paymentDetails.put(MOLPayActivity.mp_bill_name, "billname");
        paymentDetails.put(MOLPayActivity.mp_bill_email, "email@domain.com");
        paymentDetails.put(MOLPayActivity.mp_bill_mobile, "+1234567");
        paymentDetails.put(MOLPayActivity.mp_channel_editing, false); // Option to allow channel selection.
        paymentDetails.put(MOLPayActivity.mp_editing_enabled, false); // Option to allow billing information editing.
    
        // Optional for Escrow
        paymentDetails.put(MOLPayActivity.mp_is_escrow, ""); // Put "1" to enable escrow
        
        // Optional for credit card BIN restrictions
        String binlock[] = {"414170","414171"};
        paymentDetails.put(MOLPayActivity.mp_bin_lock, binlock);
        paymentDetails.put(MOLPayActivity.mp_bin_lock_err_msg, "Only UOB allowed");
    
        // For transaction request use only, do not use this on payment process
        paymentDetails.put(MOLPayActivity.mp_transaction_id, ""); // Optional, provide a valid cash channel transaction id here will display a payment instruction screen.
        paymentDetails.put(MOLPayActivity.mp_request_type, "");

## Start the payment module

    startActivityForResult(intent, MOLPayActivity.MOLPayXDK);

## Cash channel payment process (How does it work?)

    This is how the cash channels work on XDK:
    
    1) The user initiate a cash payment, upon completed, the XDK will pause at the “Payment instruction” screen, the results would return a pending status.
    
    2) The user can then click on “Close” to exit the MOLPay XDK aka the payment screen.
    
    3) When later in time, the user would arrive at say 7-Eleven to make the payment, the host app then can call the XDK again to display the “Payment Instruction” again, then it has to pass in all the payment details like it will for the standard payment process, only this time, the host app will have to also pass in an extra value in the payment details, it’s the “mp_transaction_id”, the value has to be the same transaction returned in the results from the XDK earlier during the completion of the transaction. If the transaction id provided is accurate, the XDK will instead show the “Payment Instruction" in place of the standard payment screen.
    
    4) After the user done the paying at the 7-Eleven counter, they can close and exit MOLPay XDK by clicking the “Close” button again.

## Transaction request service (Optional, NOT required for payment process)

    Step 1 - import com.molpay.molpayxdk.MOLPayService
    
    Step 2 - Prepare the Payment detail object
    
    Step 3 - MOLPayService mpservice = new MOLPayService();
    
    Step 4 - 
    mpservice.transactionRequest(paymentDetails, new MOLPayService.Callback() {
            @Override
            public void onTransactionRequestCompleted(String result) {
                // result return in json;
            }
            @Override
            public void onTransactionRequestFailed(String error) {
                // error return in json;
            }
        });

## Support

Submit issue to this repository or email to our support@molpay.com

Merchant Technical Support / Customer Care : support@molpay.com<br>
Sales/Reseller Enquiry : sales@molpay.com<br>
Marketing Campaign : marketing@molpay.com<br>
Channel/Partner Enquiry : channel@molpay.com<br>
Media Contact : media@molpay.com<br>
R&D and Tech-related Suggestion : technical@molpay.com<br>
Abuse Reporting : abuse@molpay.com