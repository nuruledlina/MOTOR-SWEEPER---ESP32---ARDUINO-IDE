/***************************
  PR440 ESP32 ROBOT SKETCH
  - ESP32 runs WiFi web server (no ESP-01 needed)
  - Autonomous mode: ultrasonic + sweeper motor
  - Manual mode: control from phone
  - LDR + 2 LEDs indicator when dark
****************************/

#include <WiFi.h>
#include <WebServer.h>

/* ------------ WiFi Config ------------ */
const char* ssid     = "yanaez";      // <<< replace
const char* password = "nohanjay";   // <<< replace

WebServer server(80);

/* ------------ Pins ------------ */
// Ultrasonic
#define trigPin 23
#define echoPin 22

// Motors (L298N)
#define left_fwd   25
#define left_rev   26
#define right_fwd  27
#define right_rev  14

// Sweeper (relay)
#define sweeperRelay 21

// LEDs & LDR
#define LDR_PIN 34
#define LED1    18
#define LED2    19

// Status
bool manualMode = false;
bool robotEnabled = true;
bool sweeperEnabled = true;

/* ------------ Autonomous Vars ------------ */
int limit_distance = 20;
long durationVal;
int actual_distance;

/* ------------ HTML Control Pad ------------ */
String htmlPage() {
  return R"rawliteral(
<!doctype html>
<html>
<head><meta name='viewport' content='width=device-width, initial-scale=1'/>
<title>PR440 Control</title>
<style>
body{font-family:Arial;text-align:center;}
button{width:110px;height:50px;font-size:16px;margin:6px;}
.dir{width:70px;height:70px;font-size:18px;}
</style></head><body>
<h3>PR440 Control Pad</h3>
<div>
<button onclick="location.href='/ROBOTON'">ROBOT ON</button>
<button onclick="location.href='/ROBOTOFF'">ROBOT OFF</button></div>
<div>
<button onclick="location.href='/SWEEPON'">SWEEP ON</button>
<button onclick="location.href='/SWEEPOFF'">SWEEP OFF</button></div>
<hr>
<div style='margin:10px;'>
<div><button class='dir' onclick="location.href='/FORWARD'">&#9650;</button></div>
<div><button class='dir' onclick="location.href='/LEFT'">&#9664;</button>
<button class='dir' onclick="location.href='/STOP'">STOP</button>
<button class='dir' onclick="location.href='/RIGHT'">&#9654;</button></div>
<div><button class='dir' onclick="location.href='/BACKWARD'">&#9660;</button></div>
</div>
<div style='margin-top:10px;'><button onclick="location.href='/AUTO'">AUTO MODE</button></div>
</body></html>
)rawliteral";
}

/* ------------ Motor helpers ------------ */
void forward() {
  digitalWrite(left_fwd, HIGH);
  digitalWrite(left_rev, LOW);
  digitalWrite(right_fwd, HIGH);
  digitalWrite(right_rev, LOW);
}

void backward() {
  digitalWrite(left_fwd, LOW);
  digitalWrite(left_rev, HIGH);
  digitalWrite(right_fwd, LOW);
  digitalWrite(right_rev, HIGH);
}

void turnLeft() {
  digitalWrite(left_fwd, LOW);
  digitalWrite(left_rev, HIGH);
  digitalWrite(right_fwd, HIGH);
  digitalWrite(right_rev, LOW);
}

void turnRight() {
  digitalWrite(left_fwd, HIGH);
  digitalWrite(left_rev, LOW);
  digitalWrite(right_fwd, LOW);
  digitalWrite(right_rev, HIGH);
}

void stopMotors() {
  digitalWrite(left_fwd, LOW);
  digitalWrite(left_rev, LOW);
  digitalWrite(right_fwd, LOW);
  digitalWrite(right_rev, LOW);
}

/* ------------ Ultrasonic autonomous ------------ */
void ultrasonicAuto() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  durationVal = pulseIn(echoPin, HIGH, 30000);
  if (durationVal == 0) actual_distance = 999;
  else actual_distance = durationVal * 0.034 / 2;
  
  if (!robotEnabled) {
    stopMotors();
    digitalWrite(sweeperRelay, LOW);
    return;
  }
  
  if (actual_distance > limit_distance) {
    digitalWrite(sweeperRelay, sweeperEnabled ? HIGH : LOW);
    forward();
  } else {
    stopMotors();
    digitalWrite(sweeperRelay, LOW);
    delay(400);
    backward();
    delay(800);
    stopMotors();
    delay(300);
    turnRight();
    delay(800);
    stopMotors();
  }
}

/* ------------ LDR LEDs ------------ */
void ldrUpdate() {
  int ldrVal = analogRead(LDR_PIN);
  const int threshold = 1800; // adjust if needed
  if (ldrVal < threshold) {
    digitalWrite(LED1, HIGH);
    digitalWrite(LED2, HIGH);
  } else {
    digitalWrite(LED1, LOW);
    digitalWrite(LED2, LOW);
  }
}

/* ------------ Command handlers ------------ */
void handleRoot() { server.send(200, "text/html", htmlPage()); }
void handleCommand(String cmd) {
  if (cmd == "ROBOTON") robotEnabled = true;
  else if (cmd == "ROBOTOFF") { robotEnabled = false; stopMotors(); digitalWrite(sweeperRelay, LOW); }
  else if (cmd == "SWEEPON") { sweeperEnabled = true; digitalWrite(sweeperRelay, HIGH); }
  else if (cmd == "SWEEPOFF") { sweeperEnabled = false; digitalWrite(sweeperRelay, LOW); }
  else if (cmd == "AUTO") manualMode = false;
  else if (cmd == "FORWARD") { manualMode = true; if(robotEnabled) forward(); }
  else if (cmd == "BACKWARD") { manualMode = true; if(robotEnabled) backward(); }
  else if (cmd == "LEFT") { manualMode = true; if(robotEnabled) turnLeft(); }
  else if (cmd == "RIGHT") { manualMode = true; if(robotEnabled) turnRight(); }
  else if (cmd == "STOP") { manualMode = true; stopMotors(); }
  server.send(200, "text/html", htmlPage());
}

/* ------------ Setup ------------ */
void setup() {
  Serial.begin(115200);
  
  // Pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(left_fwd, OUTPUT);
  pinMode(left_rev, OUTPUT);
  pinMode(right_fwd, OUTPUT);
  pinMode(right_rev, OUTPUT);
  pinMode(sweeperRelay, OUTPUT);
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  
  stopMotors();
  digitalWrite(sweeperRelay, LOW);
  
  // WiFi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  Serial.print("IP address: "); Serial.println(WiFi.localIP());
  
  // Routes
  server.on("/", handleRoot);
  server.on("/ROBOTON", [](){handleCommand("ROBOTON");});
  server.on("/ROBOTOFF", [](){handleCommand("ROBOTOFF");});
  server.on("/SWEEPON", [](){handleCommand("SWEEPON");});
  server.on("/SWEEPOFF", [](){handleCommand("SWEEPOFF");});
  server.on("/AUTO", [](){handleCommand("AUTO");});
  server.on("/FORWARD", [](){handleCommand("FORWARD");});
  server.on("/BACKWARD", [](){handleCommand("BACKWARD");});
  server.on("/LEFT", [](){handleCommand("LEFT");});
  server.on("/RIGHT", [](){handleCommand("RIGHT");});
  server.on("/STOP", [](){handleCommand("STOP");});
  
  server.begin();
}

/* ------------ Loop ------------ */
void loop() {
  server.handleClient();
  ldrUpdate();
  if (!manualMode) ultrasonicAuto();
}
