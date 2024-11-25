Usage:

```c++
enum mqtt_state_t {
  ST_POLL,
  ST_POLL_DATA
};

mqtt_state_t mqtt_state;
int mqtt_message_len;
uint8_t mqtt_buff[4];
uint8_t *mqtt_buff_end;
uint8_t *mqtt_buff_expected;
uint8_t *mqtt_buff_curr;

// ...

void setup() {
// ...
  mqtt_buff_end = mqtt_buff + sizeof(mqtt_buff);
  mqtt_state = ST_POLL;
// ...
}

void loop() {
// ...
  if (!mqtt.connected()) {
    mqtt_state = ST_POLL;
    return;
  }

  switch (mqtt_state) {
  case ST_POLL:
    mqtt.poll();
    mqtt_message_len = mqtt.available();
    if (mqtt_message_len <= 0) break;
    if (mqtt_message_len <= mqtt_buff_end - mqtt_buff) {
      mqtt_buff_expected = mqtt_buff + mqtt_message_len;
    } else {
      mqtt_buff_expected = mqtt_buff_end;
    }
    mqtt_buff_curr = mqtt_buff;
    mqtt_state = ST_POLL_DATA;
    // fallthrough
  case ST_POLL_DATA:
    if (!mqtt.pollData(&mqtt_buff_curr, mqtt_buff_expected)) {
      if (mqtt_buff_curr == mqtt_buff_expected) {
        // ...handle the new message...
        Serial.print("MQTT message (");
        Serial.print(mqtt_message_len);
        Serial.print(" bytes");
        if (mqtt_message_len > mqtt_buff_end - mqtt_buff - 1) {
          Serial.print(", truncated");
          mqtt_buff_end[-1] = 0;
        } else {
          mqtt_buff[mqtt_message_len] = 0;
        }
        Serial.print("): ");
        Serial.println((char *)mqtt_buff);
      }
      mqtt_state = ST_POLL;
    }
    break;
  }
// ...
}
```
