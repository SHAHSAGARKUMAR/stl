val uuid = ParcelUuid.fromString("0000b81d-000-1000-8000-00805f9b34fb")
    val data = byteArrayOf(66, 9, 2)
    val SAMSUNG_BLE_MANUFACTURE_ID = 0x75

    val BUDS_PLUS = 1
    val BUDS_3 = 2

    val budsPlusServiceData: ByteBuffer = ByteBuffer.allocate(24).apply {
        // 0. ver
        put(0x42.toByte())
        // 1. service ID
        put(0x09.toByte())

        // Service
        // 2. Version
        put(0x81.toByte())
        // 3. Service ID
        put(0x02.toByte())
        // 4. Feature
        put(0x14.toByte())
        // 5. Feature - Device type
        put(0x15.toByte())
        // 6. Feature - Icon
        put(0x01.toByte())
        // 7. Feature - Status
        put(0x21.toByte())
        // 8. Feature - Status - IC format
        put(0x01.toByte())
        // 9. Feature - Account count
        put(0x88.toByte())
        // 10. Feature ADV sequence NO
        put(0x01.toByte())
        // 11. Feature - Associated Data - Len
        put(0x17.toByte())
        // 12. Feature - Associated Data - Device ID
        put(0x01.toByte())
        put(0x03.toByte())
        // 14. Feature - Associated Data - Device Type
        put(0x06.toByte())
        // 15. Feature - Associated Data - Extra Features
        put(0x3C.toByte())
        // 16. Account Hash 1
        put(0x00.toByte())
        put(0x00.toByte())
        // 18. Account Hash 2
        put(0x00.toByte())
        put(0x00.toByte())
        // 20. Account Hash 3
        put(0x00.toByte())
        put(0x00.toByte())
        // 22. IC Device ID 1
        put(0x00.toByte())
        // 23. IC Device ID 2
        put(0x00.toByte())
    }

    val budsPlusScanResponse: ByteBuffer = ByteBuffer.allocate(13).apply {
// 24. IC Device ID 3
        put(0x00.toByte())
// 25. BT Mac addr
        put(0x66.toByte())
        put(0x55.toByte())
        put(0x44.toByte())
        put(0x33.toByte())
        put(0x22.toByte())
        put(0x11.toByte())
// 31. Battery Case
        put(0xff.toByte())
// 32. Battery Left
        put(0xff.toByte())
// 33. Battery Right
        put(0xff.toByte())
// 34. TX power
        put(0x00.toByte())
// 35. Prefix
        put(0x01.toByte())
// 36. Postfix
        put(0x00.toByte())

    }

    val buds3ServiceData: ByteBuffer = ByteBuffer.allocate(24).apply {
        // 0. ver
        put(0x42.toByte())
        // 1. service ID
        put(0x09.toByte())

        // Service
        // 2. Version
        put(0x81.toByte())
        // 3. Service ID
        put(0x02.toByte())
        // 4. Feature
        put(0x14.toByte())
        // 5. Feature - Device type
        put(0x15.toByte())
        // 6. Feature - Icon
        put(0x01.toByte())
        // 7. Feature - Status
        put(0x21.toByte())
        // 8. Feature - Status - IC format
        put(0x01.toByte())
        // 9. Feature - Account count
        put(0x88.toByte())
        // 10. Feature ADV sequence NO
        put(0x01.toByte())
        // 11. Feature - Associated Data - Len
        put(0x17.toByte())
        // 12. Feature - Associated Data - Device ID
        put(0x01.toByte())
        put(0x54.toByte())
        // 14. Feature - Associated Data - Device Type
        put(0x06.toByte())
        // 15. Feature - Associated Data - Extra Features
        put(0x3C.toByte())
        // 16. Account Hash 1
        put(0x91.toByte())
        put(0x5a.toByte())
        // 18. Account Hash 2
        put(0x00.toByte())
        put(0x00.toByte())
        // 20. Account Hash 3
        put(0x00.toByte())
        put(0x00.toByte())
        // 22. IC Device ID 1
        put(0x00.toByte())
        // 23. IC Device ID 2
        put(0x00.toByte())
    }

    val buds3ScanResponse: ByteBuffer = ByteBuffer.allocate(13).apply {
// 24. IC Device ID 3
        put(0x00.toByte())
// 25. BT Mac addr
        put(0x66.toByte())
        put(0x55.toByte())
        put(0x44.toByte())
        put(0x33.toByte())
        put(0x22.toByte())
        put(0x11.toByte())
// 31. Battery Case
        put(0xff.toByte())
// 32. Battery Left
        put(0xff.toByte())
// 33. Battery Right
        put(0xff.toByte())
// 34. TX power
        put(0x00.toByte())
// 35. Prefix
        put(0x03.toByte())
// 36. Postfix
        put(0x14.toByte())

    }



class BLEAdvertiser(
    private val bluetoothLeAdvertiser: BluetoothLeAdvertiser?,
    private val advertiseCallback: AdvertiserCallback
) {
    private var currentAdvertisingSet: AdvertisingSet? = null

    fun startAdvertising(type: Int) {
        if (currentAdvertisingSet != null) {
            Log.i(TAG, "Already advertising, not starting a new advertisement.")
            return
        }
        if (bluetoothLeAdvertiser == null) {
            Log.i(TAG, "Bluetooth Le Advertiser not supported")
            return
        }
        Log.i(TAG, "Advertising Starting....")
        val parameters = buildAdvertiseParameters()
        val data = buildAdvertiseData(type)
        val scanResponse = buildScanResponse(type)

        bluetoothLeAdvertiser.startAdvertisingSet(
            parameters,
            data,
            scanResponse,
            null,
            null,
            advertiseCallback
        )
        Log.i(TAG, "Advertising Started")
    }

    private fun buildAdvertiseParameters(): AdvertisingSetParameters? {
        return AdvertisingSetParameters.Builder()
            .setLegacyMode(true)
            .setScannable(true)
            .setInterval(AdvertisingSetParameters.INTERVAL_HIGH)
            .setTxPowerLevel(AdvertisingSetParameters.TX_POWER_MEDIUM)
            .build()
    }

    private fun buildAdvertiseData(type: Int): AdvertiseData {

        val serviceData = getServiceData(type)

        return AdvertiseData.Builder()
            .setIncludeDeviceName(true) // Include the device name
            .addManufacturerData(SAMSUNG_BLE_MANUFACTURE_ID, serviceData.array())
            .setIncludeTxPowerLevel(true)
            .build()
    }

    private fun getServiceData(type: Int): ByteBuffer {
        if (type == Constants.BUDS_PLUS) {
            return Constants.budsPlusServiceData
        } else {
            return Constants.buds3ServiceData
        }
    }

    private fun buildScanResponse(type: Int): AdvertiseData {

        val scanResponse = getScanResponse(type)

        return AdvertiseData.Builder()
            .addManufacturerData(SAMSUNG_BLE_MANUFACTURE_ID, scanResponse.array())
            .setIncludeDeviceName(true).build()

    }

    private fun getScanResponse(type: Int): ByteBuffer {
        if (type == Constants.BUDS_PLUS) {
            return Constants.budsPlusScanResponse
        } else {
            return Constants.buds3ScanResponse
        }
    }

    fun stopAdvertising() {
        if (currentAdvertisingSet != null) {
            bluetoothLeAdvertiser?.stopAdvertisingSet(advertiseCallback)
            currentAdvertisingSet = null
            Log.i(TAG, "Advertising Stopped")
        }
    }

    fun updateCurrentAdvertisingSet(advertisingSet: AdvertisingSet?) {
        currentAdvertisingSet = advertisingSet
    }

    companion object {
        private const val TAG = "BLEAdvertisement"
    }
    
