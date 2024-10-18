// Device configurations with corrected wattage values
let devices = [
    {ip: '192.168.178.99', wattage: 60, min_runtime: 10, last_switch_time: 0, is_on: false},
    {ip: '192.168.178.98', wattage: 75, min_runtime: 10, last_switch_time: 0, is_on: false},
    {ip: '192.168.178.74', wattage: 200, min_runtime: 1800, last_switch_time: 0, is_on: false}
    // {ip: '192.168.178.96', wattage: 300, min_runtime: 1800, last_switch_time: 0, is_on: false} // Deactivated
];

// Shelly 3EM IP address for total energy measurement
let shelly_3em_ip = '192.168.178.51';

let negative_consumption_start_time = null;
let delay_time = 30 * 1000; // 30 seconds delay in milliseconds

// Simulation Options
let bad_weather = false;  // Toggle to simulate bad weather. Set to 'true' to simulate bad weather conditions.
let wattage_offset = 0;   // Adjustable wattage offset. Use negative values to simulate reduced production or increased consumption. 
                          // Use positive values to simulate increased production or reduced consumption.

// Function to get current time in milliseconds
let get_current_time_ms = function() {
    return Math.floor(Date.now()); // Get time in milliseconds
};

// Function to handle result object
function handleResult(result, onSuccess, onError) {
    if (result && result.code !== undefined) {
        if (result.code === 200) {
            onSuccess(result);
        } else {
            onError(result);
        }
    } else {
        print('Result is undefined or does not have a "code" property:', result);
        onError(result);
    }
}

// Function to get combined energy measurement from Shelly 3EM
let get_combined_energy = function(callback) {
    let url = 'http://' + shelly_3em_ip + '/status';
    Shelly.call("HTTP.GET", {url: url}, function(result) {
        handleResult(result, function(result) {
            let data = JSON.parse(result.body);
            let total_power = data.total_power;

            // Apply wattage offset
            total_power += wattage_offset;
            print('Total Power: ' + total_power); // Debug: Print the total power fetched from Shelly 3EM with offset applied

            // Simulate bad weather by reducing total power
            if (bad_weather) {
                total_power -= 200;  // Adjust this value to simulate bad weather scenario
                print('Bad weather simulated, Total Power adjusted: ' + total_power);
            }

            callback(total_power);
        }, function() {
            print('Failed to fetch energy data, returning 0'); // Debug: Error occurred in fetching data
            callback(0);  // Return 0 if unable to fetch
        });
    });
};

// Function to control Shelly devices (turn on/off)
let control_device = function(device, action) {
    let url = 'http://' + device.ip + '/relay/0?turn=' + action;
    Shelly.call("HTTP.GET", {url: url}, function(result) {
        handleResult(result, function() {
            let now = get_current_time_ms();
            if (action === 'on') {
                device.is_on = true;
                device.last_switch_time = now;
            } else {
                device.is_on = false;
            }
            print('Device ' + device.ip + ' (' + device.wattage + 'W) turned ' + action + ' at ' + now); // Debug: Log the action
        }, function() {
            print('Failed to control device ' + device.ip + ' (' + device.wattage + 'W) for action: ' + action); // Debug: Error in controlling device
        });
    });
};

// Function to check if a device should be turned off
let should_turn_off = function(device) {
    let now = get_current_time_ms();
    let elapsed_time = (now - device.last_switch_time) / 1000; // Convert to seconds
    print('Checking if Device ' + device.ip + ' (' + device.wattage + 'W) should be turned off. Elapsed Time: ' + elapsed_time); // Debug: Time elapsed for the device
    return device.is_on && (elapsed_time >= device.min_runtime);
};

// Custom function to sort devices by wattage (ascending order)
let sort_devices_by_wattage = function(devices) {
    for (let i = 0; i < devices.length - 1; i++) {
        for (let j = 0; j < devices.length - i - 1; j++) {
            if (devices[j].wattage > devices[j + 1].wattage) {
                let temp = devices[j];
                devices[j] = devices[j + 1];
                devices[j + 1] = temp;
            }
        }
    }
};

// Main function to check energy and control devices
Timer.set(4000 /* 4 sec */, true /* repeat */, function() { // Set the timer to 4 seconds
    get_combined_energy(function(total_power) {
        let now = get_current_time_ms();
        print('Total power: ' + total_power + 'W at ' + new Date(now).toISOString());  // Debug: Log the total power in ISO format

        let current_consumption = 0;
        devices.forEach(function(device) {
            if (device.is_on) {
                current_consumption += device.wattage;
            }
        });

        if (total_power < 0) {
            if (negative_consumption_start_time === null) {
                negative_consumption_start_time = now;
                print('Negative consumption detected, starting timer'); // Debug: Starting the negative consumption timer
            }

            // Check if the negative consumption has persisted for at least 1 minute
            if ((now - negative_consumption_start_time) / 1000 >= 60) {
                // Calculate the excess solar energy
                let excess_energy = Math.abs(total_power);
                print('Negative consumption persisted for 1 minute. Excess Energy: ' + excess_energy); // Debug: Negative consumption persisted

                // Sort devices by wattage
                sort_devices_by_wattage(devices);

                // Try to turn on devices to consume as much of the excess energy as possible
                devices.forEach(function(device) {
                    if (!device.is_on && device.wattage <= excess_energy) {
                        print('Attempting to turn on Device ' + device.ip + ' (' + device.wattage + 'W). Excess Energy before: ' + excess_energy + 'W');
                        control_device(device, 'on');
                        excess_energy -= device.wattage;
                        print('Excess energy after turning on ' + device.ip + ' (' + device.wattage + 'W): ' + excess_energy + 'W'); // Debug: New excess energy level
                    }
                });
            } else {
                print('Negative consumption ongoing, but not yet 1 minute'); // Debug: Waiting for negative consumption persistence
            }
        } else {
            negative_consumption_start_time = null;  // Reset timing if consumption is positive
            print('Positive consumption detected, resetting negative consumption timer'); // Debug: Reset the negative consumption timer

            devices.forEach(function(device) {
                if (device.is_on && should_turn_off(device)) {
                    // Check if turning this device off can enable a higher wattage device
                    let new_consumption = current_consumption - device.wattage;
                    let higher_device_turned_on = false;
                    devices.forEach(function(higher_device) {
                        if (!higher_device.is_on && higher_device.wattage + new_consumption <= total_power && !higher_device_turned_on) {
                            print('Switching off Device ' + device.ip + ' (' + device.wattage + 'W) to turn on Device ' + higher_device.ip + ' (' + higher_device.wattage + 'W)');
                            control_device(device, 'off');
                            control_device(higher_device, 'on');
                            new_consumption += higher_device.wattage;
                            higher_device_turned_on = true; // Prevent multiple higher wattage devices from turning on
                            print('Switching ' + device.ip + ' off to turn on ' + higher_device.ip + '. New consumption: ' + new_consumption + 'W'); // Debug: Switched devices
                        }
                    });

                    // Check if the device has been on despite positive combined energy for `delay_time` seconds
                    let elapsed = now - device.last_switch_time;
                    if (elapsed >= delay_time + device.min_runtime * 1000 && !higher_device_turned_on) {
                        print('Device ' + device.ip + ' (' + device.wattage + 'W) has been on for required time, turning off'); // Debug: Turning off device
                        control_device(device, 'off');
                    } else {
                        print('Device ' + device.ip + ' (' + device.wattage + 'W) needs to stay on for a bit longer'); // Debug: Device needs to run longer
                    }
                } else {
                    print('Device ' + device.ip + ' (' + device.wattage + 'W) is not eligible to be turned off yet.');
                }
            });
        }
    });
});
