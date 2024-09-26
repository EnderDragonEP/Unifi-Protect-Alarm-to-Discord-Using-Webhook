# Unifi Protect Alarm to Discord Using Webhook

> This article will teach you how to post Unifi Protect alerts to Discord using Webhook
> [name=John @ 崑山科技大學 光達實驗室 KSU Eilidar Lab][time=Thursday, September 26, 2024]

## Why?

Because I want to receive notifications on my computer without using an Android emulator to install and run the Unifi Protect app.

I spent all night researching this because my friend, who is ten times smarter than I am, keeps calling me "dumb," "lazy," and "useless" :D

I hope this information provides what you need and saves you some sleep!

## How?

We need to set up a "UniFi Protect to Discord webhook data conversion server" on a computer using Python to reroute the alert data to Discord.

## 1) Get your webhook link from Discord

- In Discord, navigate to your `Server Settings` -> `Apps` -> `Integrations` -> `Webhooks`, and click on `New Webhook`
- Change the profile picture, name, and the channel where you want to post your message by selecting the appropriate channel
- Copy the webhook URL by clicking on `Copy Webhook URL`

## 2) Set up a python server

- Create a Python file and name it whatever you prefer
- Open the Python file with your favorite text editor
- Copy and paste the code below into the file

```py=
from flask import Flask, request, jsonify
import requests
from datetime import datetime

app = Flask(__name__)

# Replace this with your Discord webhook URL
DISCORD_WEBHOOK_URL = 'YOUR_WEBHOOK_URL'

# Device ID to device name mapping
# Left column is the MAC adders of the device, and the right column is the name you want to convert it to.
DEVICE_NAME_MAPPING = {
    'CAMERA_MAC_ADDERS': 'CAMERA_NAME',
    'CAMERA_MAC_ADDERS': 'CAMERA_NAME',
    'CAMERA_MAC_ADDERS': 'CAMERA_NAME'
}

@app.route('/data', methods=['POST'])
def data():
    if request.is_json:
        data = request.get_json()

        # Extract necessary fields from the received data
        if 'alarm' in data:
            alarm = data['alarm']
            triggers = alarm.get('triggers', [])

            # Access the timestamp from the root of the JSON (not inside the alarm object)
            timestamp = data.get('timestamp', None)

            # Convert timestamp to human-readable format
            readable_timestamp = convert_timestamp(timestamp)

            # Prepare data to send to the create_discord_embed function
            for trigger in triggers:
                key = trigger.get('key', 'Unknown trigger')
                device = trigger.get('device', 'Unknown device')

                device_name = DEVICE_NAME_MAPPING.get(device, device)
                # Call create_discord_embed with the key, device, and readable timestamp
                discord_message = create_discord_embed(key, device_name, readable_timestamp)

                # Send the embed to Discord
                response = post_to_discord(discord_message)

            return jsonify({"message": "JSON received and posted to Discord"}), 200
        else:
            return jsonify({"error": "Invalid JSON structure"}), 400
    else:
        return jsonify({"error": "Request must be JSON"}), 400

def post_to_discord(embed_message):
    # Send the embed to Discord webhook
    response = requests.post(DISCORD_WEBHOOK_URL, json=embed_message)

    if response.status_code == 204:
        return "Success"
    else:
        return f"Failed to post to Discord. Status code: {response.status_code}"

def create_discord_embed(trigger, device, timestamp):
    # Create an embed for Discord
    embed = {
        "embeds": [
            {
                "title": "Alert Triggered :camera_with_flash: ",
                "color": 16711680,  # Red color
                "fields": [
                    {
                        "name": "Trigger",
                        "value": trigger,
                        "inline": True
                    },
                    {
                        "name": "Device",
                        "value": device,
                        "inline": True
                    },
                    {
                        "name": "Timestamp",
                        "value": timestamp,
                        "inline": False
                    }
                ],
                "footer": {
                    "text": "Unifi Protect",
                    "icon_url": "https://pbs.twimg.com/profile_images/1610157462321254402/tMCv8T-y_400x400.png"  # Replace with your own icon URL
                },
                "timestamp": datetime.utcnow().isoformat()  # Adds current time in ISO format for Discord
            }
        ]
    }
    return embed

def convert_timestamp(timestamp):
    # Convert the timestamp to a human-readable format
    if timestamp:
        try:
            # Convert from milliseconds to seconds and format
            dt_object = datetime.utcfromtimestamp(timestamp / 1000)
            return dt_object.strftime('%Y-%m-%d %H:%M:%S UTC')
        except Exception as e:
            return "Invalid timestamp"
    return "No timestamp available"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

- Edit `DISCORD_WEBHOOK_URL` at line 8 to include the Discord webhook link that you obtained in step one
- Edit `DEVICE_NAME_MAPPING` at line 12; the left column should contain your camera's MAC address without the colons (:), and the right column should be the name you want it to display. You can add as many entries as you like
- Change the `port` at line 107 to any available port that is not being used by another process
- Start the server with Python using the following command:

  ```shell
  python3 ./<your python file name>.py
  ```

## 3) Add a webhook action in Unifi Protect

- Go to your UniFi Protect Alarm panel
- Create an alarm and give it a name
- Select what you want the alarm to send to the webhook
- Change the action from `Send Notification` to `Webhook`
- Select `Custom Webhook`
- Enter the IP address of the computer hosting the Python server created in step two into the `Delivery URL` input field like this: `http://<your_ip>:<your_port>/data`
- Check `Advanced Settings` and change the method to `POST`
- Test it out, and you're all done!

![message example](https://hackmd.io/_uploads/ryvEsbGAA.png)

## Where to find me?

- Email: <enderdragonep@gmail.com>
- Discord: noobienoodle89

###### tags: Unifi, Unifi Protect, Discord, Webhook
