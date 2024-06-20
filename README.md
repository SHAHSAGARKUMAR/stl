ParcelUuid uuid = ParcelUuid.fromString("0000fef3-0000-1000-8000-00805f9b34fb");
    String svcData = "0x23 0x50 0x4c 0x5a 0x59 0x11 0x32 0xe9 0x49 0xc4 0x8d 0x45 0xc9 0xbf 0x23 0xa0 0xf7 0x5a 0x89 0x48 0x06 0x03 0xeb";
    String manuData = "0x42 0x09 0x81 0x02 0x94 0x15 0x05 0x21 0x01 0x88 0x3d 0x0c 0x01 0x4d 0x06 0x3c 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x1b 0x00 0xed 0xd1 0x30 0x80 0x41 0x64 0x64 0x00 0x03 0x14";
    AdvertiseData data = (new AdvertiseData.Builder())
            .setIncludeDeviceName(true)
            .addManufacturerData(117, manuData.getBytes(StandardCharsets.UTF_8))
            .setIncludeDeviceName(true)
            .addServiceData(uuid, svcData.getBytes(StandardCharsets.UTF_8))
            .setIncludeTxPowerLevel(true)
            .build();
