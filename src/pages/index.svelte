<script>
    // @ts-ignore
    import { Geolocation } from '@capacitor/geolocation';
    import { BleClient, numberToUUID } from '@capacitor-community/bluetooth-le';
    import { Device } from '@capacitor/device';
    import { Keyboard } from '@capacitor/keyboard';
    import { StatusBar } from '@capacitor/status-bar';
    import { Camera, CameraResultType } from '@capacitor/camera';

    let info = new Object;
    let battery_info = {};
    let src = "";
    let deviceID = {};

    async function getDeviceInfo(){
        info = await Device.getInfo();
        deviceID = await Device.getId();
    };

    getDeviceInfo();

    async function getBatteryInfo(){
        battery_info = await Device.getBatteryInfo();
    };

    getBatteryInfo();

    let Bstatus = null;
    BleClient.initialize();

    async function getBluetoothStatus(){
        const res = await BleClient.isEnabled();
        Bstatus = res;
    }

    async function connectBluetooth(){
        await BleClient.initialize();
        const res1 = await BleClient.requestDevice();
        const isbond = await BleClient.isBonded(res1.deviceId);
        if(!isbond){
            await BleClient.createBond(res1.deviceId);
        }
        const res = await BleClient.connect(res1.deviceId, (deviceID) => onDisconnect(deviceID), {timeout:1000000000}); 
        alert(res)
    }

    function onDisconnect(deviceId){
        console.log(`device ${deviceId} disconnected`);
    }

    async function onBluetooth(){
        await BleClient.enable();
    }

    async function offBluetooth(){
        await BleClient.disable();
    }

    async function showStatus(){
        await StatusBar.show();
        await StatusBar.setBackgroundColor({ color: '#FF0000' });
    }
    

    let loc = null;
    async function getCurrentPosition(){
        const res = await Geolocation.getCurrentPosition()
        loc = res
    }

    const takePicture = async () => {

        const image = await Camera.getPhoto({
            quality: 90,
            allowEditing: true,
            resultType: CameraResultType.Uri
        });

        src = image.webPath;
    };

    var name = "svelte"
    var image = "svelte_cap.png"
</script>

<div class="main">
    <img alt="svelte_img" src={image}/>
    <h1>Welcome to {name}!</h1>

    {JSON.stringify(info)}
    <br>
    {JSON.stringify(deviceID)}
    <br>
    <p>Device Name : {info['name']}</p>
    <p>Device Id : {deviceID['uuid']}</p>

    {JSON.stringify(battery_info)}

    <h1>Geolocation</h1>
    <p>Your location is:</p>
    <p>Latitude: {loc?.coords.latitude}</p>
    <p>Longitude: {loc?.coords.longitude}</p>
  
    <button on:click={getCurrentPosition}>
      Get Current Location
    </button>

    <p>Bluetooth status: {Bstatus}</p>
    <button on:click={getBluetoothStatus}>
        Get Bluetooth Status
    </button>

    <button on:click={connectBluetooth}>
        Connect Bluetooth
    </button>

    <button on:click={onBluetooth}>
        On Blutooth
    </button>

    <button on:click={offBluetooth}>
        Off Blutooth
    </button>

    <button on:click={()=>Keyboard.show()}>
        Show Keyboard
    </button>

    <button on:click={()=>Keyboard.hide()}>
        Hide Keyboard
    </button>

    <button on:click={showStatus}>
        Show Status
    </button>

    <button on:click={()=>StatusBar.hide()}>
        Hide Status
    </button>

    <button on:click={takePicture}>
        Take picture
    </button>

    {#if src !== ''}
        <div>
            {src}
            <img {src} alt="cam_img" id="camImage"/>
        </div>
    {/if}
</div>

<style>

    .main {
        text-align: center;
        margin: 50px auto;
        word-break: break-all;
    }

    img {
        max-width: 100%;
    }
</style>