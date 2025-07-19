# SecPlusGDO Component Details

## YAML Configuration Options

| Component | Option | Type | Req/Opt | Default | Description |
|-----------|--------|------|---------|---------|-------------|
| **secplus_gdo** | `output_gdo_pin` | GPIO Out | Req | 1 | UART TX pin for GDO communication. This pin forms the security+ wireline transmit for v1 and v2 protocols and also drives a dry contact door control switch. |
| **secplus_gdo** | `input_gdo_pin` | GPIO In | Req | 2 | UART RX pin for GDO communication. This pin forms the wireline receive for v1 and v2 protocols. |
| **secplus_gdo** | `tof_sda_pin` | GPIO In | Opt | - | Time-of-flight sensor SDA pin optional park presence sensor input data control signal. |
| **secplus_gdo** | `tof_scl_pin` | GPIO In | Opt | - | Time-of-flight sensor SCL pin optional park presence sensor data clock signal. |
| **secplus_gdo** | `input_obst_pin` | GPIO In | Opt | - | Obstruction sensor input pin. When defined the component will use the optical obstruction sensor signal to detect an obstruction by counting pulses from it. |
| **secplus_gdo** | `dc_open_pin` | GPIO In | Opt | - | Dry contact open pin. Serves a dry contact configuration reporting a door open switch state. |
| **secplus_gdo** | `dc_close_pin` | GPIO In | Opt | - | Dry contact close pin. Serves a dry contact configuration reporting a door closed switch sensor state. |
| **cover** | `pre_close_warning_duration` | Time | Opt | 0s | Duration for pre-close warning. Functions as an automation control setting for audio warnings. |
| **cover** | `pre_close_warning_start` | Auto | Opt | - | Automation trigger for pre-close warning start. Functions as an automation control to start an audio warning device. |
| **cover** | `pre_close_warning_end` | Auto | Opt | - | Automation trigger for pre-close warning end. Functions as an automation control to stop the audio warning device. |
| **binary_sensor** | `type` | Enum | Req | - | Sensor type: motion, obstruction, motor, button, sync, vehicle_parked, vehicle_arriving, vehicle_leaving |
| **sensor** | `type` | Enum | Req | - | Sensor type: openings, tof_distance, paired_devices_total, paired_devices_remotes, paired_devices_keypads, paired_devices_wall_controls, paired_devices_accessories |
| **number** | `type` | Enum | Req | - | Number type: open_duration, close_duration, client_id, rolling_code, min_command_interval, time_to_close, vehicle_parked_threshold, vehicle_parked_threshold_variance |
| **number** | `min_command_interval` | uint32 | Opt | 250 | Minimum command interval in milliseconds (250-1500ms). Provides a control to throttle the rate of command transmision on the wireline. Helps to prevent browning out sensitive wall units. |
| **number** | `time_to_close` | uint16 | Opt | 300 | Time to close in seconds (0-65535). Provides a control to adjust the Time to Close function of Secuity+ v2 door openers. Most door openers will accept 10 up to 600 seconds. |
| **number** | `vehicle_parked_threshold` | uint16 | Opt | 100 | Vehicle parked detection threshold (10-400). Sets the surface distance of a parked vehicle. |
| **number** | `vehicle_parked_threshold_variance` | uint16 | Opt | 5 | Vehicle parked detection variance (2-15). Provisions a control to set the variance of reflected distance signals on difficult surfaces like glass or black paint. Higher values will stabilize the sensor |
| **select** | `initial_option` | Enum | Opt | auto | Protocol selection: auto, security+1.0, security+2.0, security+1.0 with smart panel, dry contact |
| **switch** | `type` | Enum | Req | - | Switch type: learn, toggle_only, obst_override |

## Value Ranges and Units

| Component | Option | Min | Max | Step | Unit |
|-----------|--------|-----|-----|------|------|
| **number** (open_duration) | - | 0 | 240 | 0.1 | sec |
| **number** (close_duration) | - | 0 | 240 | 0.1 | sec |
| **number** (client_id) | - | 0x666 | 0x7ff666 | 1 | - |
| **number** (rolling_code) | - | 0 | 0xFFFFFF | 1 | - |
| **number** (min_command_interval) | - | 250 | 1500 | 50 | ms |
| **number** (time_to_close) | - | 0 | 65535 | 60 | sec |
| **number** (vehicle_parked_threshold) | - | 10 | 400 | 1 | - |
| **number** (vehicle_parked_threshold_variance) | - | 2 | 15 | 1 | - |

## Configuration Notes

- All components require `secplus_gdo_id` to reference the main secplus_gdo component
- Components marked with "Optional" can be omitted from configuration
- The `tof_sda_pin` and `tof_scl_pin` options enable time-of-flight sensor functionality
- The component uses specific compile-time defines based on configured options
- Duration measurements (`open_duration`, `close_duration`) are read-only values reported by the garage door opener

## Component Dependencies

- **Main Component**: `secplus_gdo` - Core component that manages the garage door opener communication
- **Dependencies**: All sub-components depend on the main `secplus_gdo` component
- **External Libraries**:
  - VL53L1 (for time-of-flight sensor functionality)
  - GDOLIB (core garage door opener library)

## Supported Binary Sensor Types

| Type | Description |
|------|-------------|
| `motion` | Motion detection from garage door opener |
| `obstruction` | Obstruction detection status |
| `motor` | Motor running status |
| `button` | Button press detection |
| `sync` | Synchronization status with garage door opener |
| `vehicle_parked` | Vehicle presence detection (requires ToF sensor) |
| `vehicle_arriving` | Vehicle arriving detection (requires ToF sensor) |
| `vehicle_leaving` | Vehicle leaving detection (requires ToF sensor) |

## Supported Sensor Types

| Type | Description | Unit |
|------|-------------|------|
| `openings` | Total number of door openings | count |
| `tof_distance` | Time-of-flight distance measurement | mm |
| `paired_devices_total` | Total paired devices | count |
| `paired_devices_remotes` | Number of paired remotes | count |
| `paired_devices_keypads` | Number of paired keypads | count |
| `paired_devices_wall_controls` | Number of paired wall controls | count |
| `paired_devices_accessories` | Number of paired accessories | count |

## Supported Switch Types

| Type | Description |
|------|-------------|
| `learn` | Enable/disable learn mode for pairing devices |
| `toggle_only` | Toggle-only mode for door operation |
| `obst_override` | Override obstruction detection |
