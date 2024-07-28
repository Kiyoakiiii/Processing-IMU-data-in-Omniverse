# Processing IMU data in Omniverse


# Prerequisites
Before running any of the installation a number of prerequisites are required.

Follow the [Getting Started with Omniverse ](https://www.nvidia.com/en-us/omniverse/download/) to install the latest Omniverse version.

If you've already installed Omniverse, ensure you have updated to the latest

* Python 3.10 or greater
* Kit 105.1 or greater
* USD Composer 2023.2.0 or greater
* Nucleus 2023.1 or greater


# Installation

Once you have the latest Omniverse prerequisites installed, please run the following to install the needed Omniverse USD resolver, Omni client, and related dependencies.

```
Windows
> install.bat
```
```
Linux
> ./install.sh
```


### App Link Setup

If `app` folder link doesn't exist or becomes broken it can be recreated. For a better developer experience it is recommended to create a folder link named `app` to the *Omniverse Kit* app installed from *Omniverse Launcher*. A convenience script to use is included.

Run:

```
Windows
> link_app.bat
```
```
Linux
> ./link_app.sh
```


If successful you should see an `app` folder link in the root of this repo.

If multiple Omniverse apps are installed the script will automatically select one. Or you can explicitly pass an app:

```
Windows
> link_app.bat --app create
```
```
Linux
> ./link_app.sh --app create
```

You can also pass an explicit path to the Omniverse Kit app:

```
Windows
> link_app.bat --path "%USERPROFILE%/AppData/Local/ov/pkg/create-2023.2.0"
```
```
Linux
> ./link_app.sh --path "~/.local/share/ov/pkg/create-2023.2.0"
```

# Windows side

To execute the application run the following:
```
> python source/cube_movement/run_app.py
    -u <user name>
    -p <password>
    -s <nucleus server> (optional default: localhost)
```

Username and password are of the Nucleus instance (running on local workstation or on cloud) you will be connecting to for your projects.

The cube movement application can be found in the ./source/cube_movement folder. It will perform the following:

1.Initialize the stage
2.Open a connection to Nucleus.
3.Create a cube to omniverse://localhost/Users/<Username>/imu-data/Cube_imu_data_example/
4.Create or join a Live Collaboration Session named iot_session.
5.estimate roll pitch and yaw and calculate the current component of gravity acceleration in the xyz direction based on the transmitted acceleration and angular velocity data to eliminate the gravity acceleration.
6.Use the processed acceleration and angular velocity data to calculate the displacement and rotation and write it to the cube's motion

In USD Composer or Kit, open omniverse://localhost/Users/<Username>/imu-data/Cube_imu_data_example/Cube_imu_data_example.usd and join the iot_session live collaboration session.

Here's how-to join a live collaboration session. Press the live icon in the upper right corner and Click on Join Session

![屏幕截图 2024-07-28 163601](https://github.com/user-attachments/assets/07399f03-1268-428b-8e89-ce990acc9756)

Select iot-session from the drop down to join the already created live session.

![屏幕截图 2024-07-28 163617](https://github.com/user-attachments/assets/3b4fd994-1946-495d-85d8-d97e837a0911)

After joining the session, you can see the created blocks

![image](https://github.com/user-attachments/assets/ddd69fee-40e0-40dd-9a5f-bc2a057431a4)


# Sensor side

You can use a development board (here we use Jetson Orin Nano) to read IMU data.And we use the MPU6050 sensor to read the acceleration and angular velocity in the xyz direction.

Connect the MPU6050 and Jetson Orin Nano board according to the correct pins

![image](https://github.com/user-attachments/assets/88bb0fd6-ed0b-4097-afc1-fa7add2e1b86)

Write a program on Jetson Orin Nano to read sensor data (Flask is used here to send and receive data)
```
> import time
import board
import busio
import requests
from adafruit_mpu6050 import MPU6050
i2c = busio.I2C(board.SCL, board.SDA)
mpu = MPU6050(i2c)
while True:
    accel_x, accel_y, accel_z = mpu.acceleration
    gyro_x, gyro_y, gyro_z = mpu.gyro
    data = {
        'accel_x': round(accel_x, 2),
        'accel_y': round(accel_y, 2),
        'accel_z': round(accel_z, 2),
        'gyro_x': round(gyro_x, 2),
        'gyro_y': round(gyro_y, 2),
        'gyro_z': round(gyro_z, 2),
    }
    response = requests.post("http://"Your IP address":5000/imu", json=data)
    print("Sent: {}, Received: {}".format(data, response.status_code))
    time.sleep(The time interval for sending data)
```

After running the code on the Windows side, run this code on the development board to establish a connection and transmit IMU data to Windows.And the generated blocks will move according to the motion data of the sensor.




