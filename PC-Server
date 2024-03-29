#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 
#include <sys/types.h> 
#include <sys/socket.h> 
#include <netinet/in.h> 
#include <netdb.h> 
#include <unistd.h>
#define SERVER_PORT 5423 

#define MAX_LINE 256
void sendToggleCommand(int s, const char* led) {
  char command[MAX_LINE];
  snprintf(command, sizeof(command), "toggle %s", led); size_t len = strlen(command) + 1;
  send(s, command, len, 0); 
}

void sendReadCommand(int s, const char* commandType) { char command[MAX_LINE];
snprintf(command, sizeof(command), "read %s", commandType); size_t len = strlen(command) + 1;
send(s, command, len, 0);
}
void sendSubscribeCommand(int s, int readings, int interval) {
  char command[MAX_LINE];
  snprintf(command, sizeof(command), "subscribe %d %d", readings, interval); size_t len = strlen(command) + 1;
  send(s, command, len, 0); 
}
float celsiusToFahrenheit(float celsius) { 
  return (celsius * 9 / 5 + 32);
}
float temperatureFromADC(int adcValue, int isCelsius) {
  float temperatureCelsius = (adcValue / 4095.0) * 100.0; // Convert ADC value to Celsius
  return isCelsius ? temperatureCelsius : celsiusToFahrenheit(temperatureCelsius); 
}
void receiveData(int s, int isCelsius) {
char buffer[MAX_LINE];
int bytesReceived = recv(s, buffer, sizeof(buffer), 0);
if (bytesReceived < 0) { perror("Error receiving data"); 
// Handle error


} else if (bytesReceived == 0) { 
printf("Connection closed by server\n"); // Handle connection closure
} else {
buffer[bytesReceived] = '\0'; // Add null-terminator to received data
// Convert received string to an integer ADC value

  int receivedValue = atoi(buffer);
  // Convert received ADC value to Celsius/Fahrenheit based on mode
  float temperature = temperatureFromADC(receivedValue, isCelsius);
  printf("Received ADC value from server: %d\n", receivedValue); if (isCelsius) {
    printf("Temperature in Celsius: %.2f\n", temperature); 
  } else {
      printf("Temperature in Fahrenheit: %.2f\n", temperature); }
        }   
  }

  int connectToESP32(const char* server_ip) { struct sockaddr_in sin;
  int s;

  struct hostent *hp = gethostbyname(server_ip); if (!hp) {
  fprintf(stderr, "simplex-talk: unknown host: %s\n", server_ip); 
  exit(1);

}

  bzero((char *)&sin, sizeof(sin));
  sin.sin_family = AF_INET;
  bcopy(hp->h_addr, (char *)&sin.sin_addr, hp->h_length); sin.sin_port = htons(SERVER_PORT);
  if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) { perror("simplex-talk: socket");
  exit(1);
  }
  if (connect(s, (struct sockaddr *)&sin, sizeof(sin)) < 0) {
    perror("simplex-talk: connect"); close(s);
    exit(1);
  }
  return s; 
}

int main() {
  const char* server_ip = "10.99.155.233"; // Replace with ESP32 IP address
  int s = connectToESP32(server_ip);
  int isCelsius = 1; // Flag to keep track of temperature mode (1 for Celsius, 0 for Fahrenheit)
while (1) {
  printf("Enter command (toggle , read, mode , subscribe, exit): "); 
char buf[MAX_LINE];
  fgets(buf, sizeof(buf), stdin);
  buf[strcspn(buf, "\n")] = '\0'; // Remove trailing newline
  if (strncmp(buf, "toggle", 6) == 0) {
    char led[MAX_LINE];
    sscanf(buf, "toggle %[^\n]", led);
  if (strcmp(led, "LED1") == 0 || strcmp(led, "LED2") == 0) {
  sendToggleCommand(s, led); 
} else {
  printf("Unknown command.\n"); 
  }
} else if (strcmp(buf, "read") == 0) {
sendReadCommand(s, isCelsius ? "celsius" : "fahrenheit");
sleep(1); // Introduce a delay to allow the ESP32 to process and respond receiveData(s, isCelsius);

} else if (strncmp(buf, "mode", 4) == 0) {
  char mode[MAX_LINE];
  sscanf(buf, "mode %[^\n]", mode);
  if (strcmp(mode, "celsius") == 0 || strcmp(mode, "fahrenheit") == 0) {
    if (strcmp(mode, "celsius") == 0) { isCelsius = 1;
    } else {
      isCelsius = 0;
}
  printf("Temperature mode set to %s.\n", mode);
  send(s, buf, strlen(buf), 0); // Send mode change to ESP32
} else {
  printf("Invalid mode command. Use 'mode celsius' or 'mode fahrenheit'\n"); 
  }
} else if (strncmp(buf, "subscribe", 9) == 0) { 
  int readings, interval;
if (sscanf(buf, "subscribe %d %d", &readings, &interval) == 2) { 
  sendSubscribeCommand(s, readings, interval);
} else {
  printf("Invalid subscribe command format. Use 'subscribe <readings> <interval>'\n"); }
} else if (strcmp(buf, "exit") == 0) { 
  printf("Exiting...\n");
  send(s, buf, strlen(buf), 0); close(s);
    exit(0);
  } else {
    printf("Unknown command.\n"); }
  }
    close(s);
    return 0;
}
