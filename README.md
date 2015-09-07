####Change of Android 5.1 source to skip network validation.

* In China, we can't connect to http://clients3.google.com/generate_204.
* If you don't ship network validation, it will always search network to get Google server 204. 
* Then it causes a lower battery life and the icons of signal & wifi on status bar will contain the exclamation marks.

####Path to frameworks/base/services/core/java/com/android/server/connectivity/NetworkMonitor.java.
#####line 264
* Add: before "public NetworkMonitor(Context context, Handler handler, NetworkAgentInfo networkAgentInfo".

		//Add. In China, we can't connect to http://clients3.google.com/generate_204.
		private static boolean mSkipNetworkValidation = true;
		//End. 
 
#####line 301
* Add:   following "mCaptivePortalLoggedInResponseToken = String.valueOf(new Random().nextLong());".

		//Add. In China, we can't connect to http://clients3.google.com/generate_204.
		mSkipNetworkValidation = mContext.getResources().getBoolean(
			com.android.internal.R.bool.config_skip_network_validation);

#####line 488
* Change: "int httpResponseCode = isCaptivePortal();" to following codes.

           //M: Add. In China, we can't connect to http://clients3.google.com/generate_204.
          log("mSkipNetworkValidation ="+mSkipNetworkValidation);
           if (mSkipNetworkValidation) {
               mConnectivityServiceHandler.sendMessage(obtainMessage(EVENT_NETWORK_TESTED,
                   NETWORK_TEST_RESULT_VALID, 0, mNetworkAgentInfo));

               //CMCC mobile network sometimes redirect issue, so skip mobile CaptivePortal
               if (mNetworkAgentInfo.networkCapabilities.hasTransport(
                   NetworkCapabilities.TRANSPORT_CELLULAR)){
                   return HANDLED;
               }
          }

           int httpResponseCode = isCaptivePortal();

          if (mSkipNetworkValidation) {
              if (httpResponseCode != 204 && httpResponseCode >= 200 && httpResponseCode <= 399){
                 transitionTo(mCaptivePortalState);
              } else {
                 transitionTo(mValidatedState);
               }
               return HANDLED;
           }
           //End: Add. In China, we can't connect to http://clients3.google.com/generate_204.

####Path to frameworks/base/res/res/values/config.xml.
#####line 2215
* Add:    

		<!-- //M: Add. In China, we can't connect to http://clients3.google.com/generate_204.-->     
		<bool name="config_skip_network_validation">true</bool>


####Path to frameworks/base/res/res/values/symbols.xml.
#####line 2179
* Add:    

		<!-- //M: Add. In China, we can't connect to http://clients3.google.com/generate_204.-->     
		<java-symbol type="bool" name="config_skip_network_validation" />
