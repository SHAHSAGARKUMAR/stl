class MainActivity : ComponentActivity() {
    private lateinit var bluetoothAdapter: BluetoothAdapter
    private var bluetoothLeAdvertiser: BluetoothLeAdvertiser? = null
    private var currentAdvertisingSet: AdvertisingSet? = null

//    call back to handle advertising events

    private val REQUEST_CODE_PERMISSIONS = 1
    private val REQUIRED_PERMISSIONS = arrayOf(
        Manifest.permission.BLUETOOTH,
        Manifest.permission.BLUETOOTH_ADMIN,
        Manifest.permission.BLUETOOTH_ADVERTISE,
        Manifest.permission.ACCESS_FINE_LOCATION
    )

    private val advertiseCallback = object : AdvertisingSetCallback() {
        override fun onAdvertisingSetStarted(
            advertisingSet: AdvertisingSet?,
            txPower: Int,
            status: Int
        ) {

            when(status){
                AdvertisingSetCallback.ADVERTISE_SUCCESS ->{
                    Log.i(TAG, "Advertising started successfully. TxPower: $txPower")
                    currentAdvertisingSet = advertisingSet
                    Toast.makeText(this@MainActivity, "Advertising started", Toast.LENGTH_SHORT).show()
                }
                AdvertisingSetCallback.ADVERTISE_FAILED_ALREADY_STARTED -> {
                    Log.e(TAG, "Advertising failed to start: Already started")
                    Toast.makeText(this@MainActivity, "Advertising failed: Already started", Toast.LENGTH_SHORT).show()
                }
                AdvertisingSetCallback.ADVERTISE_FAILED_DATA_TOO_LARGE -> {
                    Log.e(TAG, "Advertising failed to start: Data too large")
                    Toast.makeText(this@MainActivity, "Advertising failed: Data too large", Toast.LENGTH_SHORT).show()
                }
                AdvertisingSetCallback.ADVERTISE_FAILED_FEATURE_UNSUPPORTED -> {
                    Log.e(TAG, "Advertising failed to start: Feature unsupported")
                    Toast.makeText(this@MainActivity, "Advertising failed: Feature unsupported", Toast.LENGTH_SHORT).show()
                }
                AdvertisingSetCallback.ADVERTISE_FAILED_INTERNAL_ERROR -> {
                    Log.e(TAG, "Advertising failed to start: Internal error")
                    Toast.makeText(this@MainActivity, "Advertising failed: Internal error", Toast.LENGTH_SHORT).show()
                }
                AdvertisingSetCallback.ADVERTISE_FAILED_TOO_MANY_ADVERTISERS -> {
                    Log.e(TAG, "Advertising failed to start: Too many advertisers")
                    Toast.makeText(this@MainActivity, "Advertising failed: Too many advertisers", Toast.LENGTH_SHORT).show()
                }
            }
        }

        override fun onAdvertisingDataSet(advertisingSet: AdvertisingSet?, status: Int) {
            Log.i(TAG, "onAdvertisingDataSet() status: $status")
        }

        override fun onScanResponseDataSet(advertisingSet: AdvertisingSet?, status: Int) {
            Log.i(TAG, "onScanResponseDataSet() status: $status")
        }

        override fun onAdvertisingSetStopped(advertisingSet: AdvertisingSet?) {
            Log.i(TAG, "onAdvertisingSetStopped()")
            Toast.makeText(this@MainActivity, "Advertising Stopped", Toast.LENGTH_SHORT).show()
        }
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BLEScreen(
                onStartAdvertising = { startAdvertising() },
                onStopAdvertising = { stopAdvertising() }
            )
        }
        if(allPermissionsGranted()){
            setupBluetooth()
        }else{
            ActivityCompat.requestPermissions(this, REQUIRED_PERMISSIONS, REQUEST_CODE_PERMISSIONS)
        }
    }

    private fun setupBluetooth() {
        val bluetoothManager = getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
        bluetoothAdapter = bluetoothManager.adapter

        if(bluetoothAdapter.isEnabled){
            bluetoothLeAdvertiser = bluetoothAdapter.bluetoothLeAdvertiser
        }
        else{
            Toast.makeText(this, "Bluetooth is not enabled", Toast.LENGTH_SHORT).show()
        }
    }


    private fun startAdvertising() {
        if(bluetoothLeAdvertiser == null){
            Log.e(TAG, "Bluetooth LE Advertising is not supported on this device")
            return
        }
        
        Log.i(TAG, "Advertising Starting....")
        val parameters = AdvertisingSetParameters.Builder()
            .setLegacyMode(false)
            .setConnectable(true)
            .setInterval(AdvertisingSetParameters.INTERVAL_HIGH)
            .setTxPowerLevel(AdvertisingSetParameters.TX_POWER_MEDIUM)
            .build()

        // set up advertising data
        val uuid =  ParcelUuid.fromString("0000fef3-0000-1000-8000-00805f9b34fb")
        val svcData = "23504c5a591132e949c48d45c9bf23a0f75a89480603eb"
        val manuData = "420981029415052101883d0c014d063c0000000000000000001b00edd13080416464000314"
        val data = AdvertiseData.Builder()
            .setIncludeDeviceName(true)
            .addManufacturerData(117, manuData.toByteArray(StandardCharsets.UTF_8))
            .addServiceData(uuid, svcData.toByteArray(StandardCharsets.UTF_8))
            .setIncludeTxPowerLevel(true)
            .build();

        // start advertising
        
        bluetoothLeAdvertiser?.startAdvertisingSet(parameters,data,null,null,null,advertiseCallback)
        Log.i(TAG, "Advertising Started")
    }

   
    private fun stopAdvertising(){
        if(currentAdvertisingSet != null){
            bluetoothLeAdvertiser?.stopAdvertisingSet(advertiseCallback)
            currentAdvertisingSet=null
            Log.i("BLE", "Advertising Stopped")
        }
    }

    private fun allPermissionsGranted(): Boolean {
        return REQUIRED_PERMISSIONS.all {
            ContextCompat.checkSelfPermission(this, it) == PackageManager.PERMISSION_GRANTED
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if(requestCode == REQUEST_CODE_PERMISSIONS){
            if(allPermissionsGranted())
                setupBluetooth()
        } else{
            Toast.makeText(this, "Permission not grated", Toast.LENGTH_SHORT).show()
            finish()
        }
    }

    companion object {
        private const val TAG = "BLEAdvertisement"
    }

}

@Composable
fun BLEScreen(onStartAdvertising: () -> Unit, onStopAdvertising: () -> Unit) {
    Column(
        modifier= Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        Button(onClick = onStartAdvertising, modifier = Modifier.fillMaxWidth()) {
            Text("Start Advertising")
        }
        Button(onClick = onStopAdvertising, modifier = Modifier.fillMaxWidth()) {
            Text("Stop Advertising")
        }
    }
}
