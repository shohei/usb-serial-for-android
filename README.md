#usb-serial-for-android
最新版のライブラリだとうまく動いてない

https://github.com/mik3y/usb-serial-for-android

```groovy
compile 'com.hoho.android:usb-serial-for-android:0.2.0-SNAPSHOT@aar'
repositories {
  maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
  mavenCentral()
}
```
## Walkthrough
researching...
```Java
import android.app.Activity;
import android.content.Context;
import android.hardware.usb.UsbDeviceConnection;
import android.hardware.usb.UsbManager;
import android.os.Bundle;
import android.util.Log;

import com.hoho.android.usbserial.driver.UsbSerialDriver;
import com.hoho.android.usbserial.driver.UsbSerialPort;
import com.hoho.android.usbserial.driver.UsbSerialProber;

import java.io.IOException;
import java.util.List;


public class MainActivity extends Activity {
    UsbManager manager;
    UsbSerialDriver driver;
    UsbDeviceConnection connection;
    UsbSerialPort port;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initSerialPort();
    }

    protected void initSerialPort() {
        manager = (UsbManager) getSystemService(Context.USB_SERVICE);
        List<UsbSerialDriver> availableDrivers = UsbSerialProber.getDefaultProber().findAllDrivers(manager);
        if (availableDrivers.isEmpty()) {
            return;
        }
        driver = availableDrivers.get(0);
        connection = manager.openDevice(driver.getDevice());
        if (connection == null) {
            return;
        }
        List<UsbSerialPort> availablePorts = driver.getPorts();
        if (availablePorts == null) {
            return;
        }
        port = availablePorts.get(0);
    }

    protected void openSerialPort() throws IOException{
        try {
            port.setParameters(9600,8,1,0);
            byte buffer[] = new byte[16];
            int numBytesRead = port.read(buffer, 1000);
            Log.d("moge", "Read " + numBytesRead + " bytes.");
        } catch (IOException e) {
            // Deal with error.
            throw new IOException();
        } finally {
            port.close();
        }
    }

    public void start_read_thread() {
        new Thread(new Runnable() {
            public void run() {
                try {
                    while (true) {
                        byte buf[] = new byte[256];
                        int num = port.read(buf, buf.length);
                        if (num > 0)
                            Log.v("arduino", new String(buf, 0, num)); 
                        Thread.sleep(10);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

}
```
