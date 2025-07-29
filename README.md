# WSHome ESP32 Smart Home Library

This is an ESP32 Smart Home system library developed for **WSHome**. It provides a seamless integration with your backend server and allows you to control devices remotely through WebSocket.

Before getting started with this project, ensure that you have the necessary dependencies in place. Specifically, you will need to download the **official ESP32 WebSocket client library** from Espressif's website.

### Prerequisites

1. **ESP32 WebSocket Client Library**
   First, download the official ESP32 WebSocket client library from the Espressif website:
   [ESP32 WebSocket Client](https://components.espressif.com/components/espressif/esp_websocket_client)

2. **Clone the WSHome ESP32 Library**
   Next, clone the following GitHub repository to your project directory:

   ```
   git clone https://github.com/L0rdL0ther/WSHome-ESP32-Library
   ```

3. **Include the Libraries in Your Project**
   After cloning the repository, include the necessary libraries in your project by modifying the `CMakeLists.txt` file in the `main/` directory:

   ```txt
   idf_component_register(SRCS "main.c" "WSHome-ESP32-Library/wifi_control.c" "WSHome-ESP32-Library/smart_home.c"
   INCLUDE_DIRS ".")
   ```

   As you can see, the two libraries `"WSHome-ESP32-Library/wifi_control.c"` and `"WSHome-ESP32-Library/smart_home.c"` are included. This allows you to use these components in your project.

---

### Setting Up the Backend

1. **Log In or Sign Up to the WSHome Backend**
   After setting up your server, log in to the WSHome platform. If you do not have an account, sign up first.

2. **Add Your ESP32 Device**
   After logging in, navigate to the **"Add ESP32"** button located at the top-right of the main screen. Here, you can give your ESP32 device a name and create it. After creation, a **token** will be provided. This token will be used for authentication.

3. **Configure the Main Project**
   Now, go to the `main.c` file of your project and include the necessary header file for smart home functionality:

   ```c
   #include "smart_home/smart_home.h"
   ```

   **Important**: You must be connected to Wi-Fi and the backend server must be on the same network for successful communication.

4. **Set Up the WebSocket URI and Token**
   Add the following definitions at the top of your project to set up the WebSocket URI and token for authentication:

   ```c
   #define AUTH_TOKEN "auth:(your provided token)"
   #define WEBSOCKET_URI "ws://(your backend address)/ws/esp32"
   ```

   **Important**: If your backend is hosted on the cloud, make sure to enter the correct cloud address.

---

### Main Application Code

In the `app_main` section of your `main.c` file, set up the initialization of the smart home system:

```c
void app_main(void) {
    // Wi-Fi connection should be established here

    smart_home_config_t smart_home_config = {
        .websocket_uri = WEBSOCKET_URI,
        .auth_token = AUTH_TOKEN,
        .auto_reconnect = true,
        .reconnect_timeout_ms = 10000
    };
    ret = smart_home_init(&smart_home_config, message_callback, NULL);
    if (ret != ESP_OK) {
        ESP_LOGE(TAG, "Smart home system initialization failed: %s", esp_err_to_name(ret));
        return;
    }
    smart_home_bind_device(102, "1"); // Initialize the device by updating it on the backend (102 is the device ID you created in the backend)

    ESP_LOGI(TAG, "System successfully initialized, waiting for commands...");
}
```

---

### Callback Function for Handling Messages

Below your `app_main` function, add the following callback function to handle messages received from the backend:

```c
// Callback function that is triggered when a message is received from Backend
static void message_callback(int device_id, control_type_t control_type,
                           const char *value, esp_websocket_client_handle_t client,
                           void *user_context) {
    ESP_LOGI(TAG, "Message received: ID=%d, Type=%d, Value=%s", device_id, control_type, value);  // Log the received message

    switch (control_type) {  // Switch case to handle different control types
        case CONTROL_TYPE_SWITCH:  // Case for handling switch control
                if (device_id == 102) {  // Check if the device ID is 102
                    bool relay_state = (strcmp(value, "0") == 0);  // If the value is "0", set relay_state to false, otherwise true
                    gpio_set_level(RELAY_GPIO, relay_state);  // Set the relay's GPIO level based on the relay_state
                    ESP_LOGI(TAG, "Relay (GPIO 19) %s", relay_state ? "ON" : "OFF");  // Log whether the relay was turned on or off
                    smart_home_bind_device(device_id, relay_state ? "0" : "1");  // Bind the device to the new state (on or off)
                } else {
                    ESP_LOGW(TAG, "Unknown device ID: %d", device_id);  // Log a warning for unknown device IDs
                }
        break;

        case CONTROL_TYPE_SLIDER:  // Case for handling slider control
                ESP_LOGW(TAG, "Slider control type not implemented yet (Device ID: %d)", device_id);  // Log a warning if the slider control is not yet implemented
        break;

        case CONTROL_TYPE_RGB_PICKER:  // Case for handling RGB picker control
                ESP_LOGW(TAG, "RGB control not implemented yet (Device ID: %d)", device_id);  // Log a warning if RGB control is not yet implemented
        break;

        default:  // Default case for unsupported control types
            ESP_LOGW(TAG, "Unsupported control type: %d (Device ID: %d)", control_type, device_id);  // Log a warning for unsupported control types
        break;
    }
}
```

### Communication Flow

Once everything is set up, the ESP32 device will be able to communicate with the backend server over WebSocket. The backend sends control commands, and the ESP32 executes the corresponding actions based on the control type.

---

### Conclusion

With these steps completed, you have successfully integrated the **WSHome ESP32 Smart Home Library** into your project. You can now control devices through the backend and receive real-time updates from the ESP32.

If you encounter any issues or have questions, feel free to open an issue in the [GitHub repository](https://github.com/L0rdL0ther/WSHome-ESP32-Library).
