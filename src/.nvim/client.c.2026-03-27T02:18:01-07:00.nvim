#define _DEFAULT_SOURCE
#define _POSIX_C_SOURCE 200809L

#include <arpa/inet.h>
#include <errno.h>
#include <pthread.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

#define PORT 8000
#define BUF_SIZE 5000
#define ADDR "127.0.0.1"

#define handle_error(msg)                                                      \
  do {                                                                         \
    perror(msg);                                                               \
    exit(EXIT_FAILURE);                                                        \
  } while (0)

struct thread_args {
  int sfd;
  FILE *log_fp;
};

// Given by the assignment
int convert(uint8_t *buf, ssize_t buf_size, char *str, ssize_t str_size) {
  if (buf == NULL || str == NULL || buf_size <= 0 ||
      str_size < (buf_size * 2 + 1)) {
    return -1;
  }

  for (int i = 0; i < buf_size; i++) {
    sprintf(str + i * 2, "%02X", buf[i]);
  }
  str[buf_size * 2] = '\0';
  return 0;
}

void *recieve_thread(void *arg) {
  struct thread_args *args = (struct thread_args *)arg;
  int sfd = args->sfd;
  FILE *log_fp = args->log_fp;

  char buf[BUF_SIZE];
  ssize_t bytes_read;
  int buf_len = 0;

  // reading broadcasted data in the server, this way we can show the server
  // threads
  while ((bytes_read = read(sfd, buf + buf_len, BUF_SIZE - buf_len)) > 0) {
    buf_len = buf_len + bytes_read;

    while (buf_len > 0) {
      char msgType = buf[0];

      // Message for clients
      if (msgType == 0) {
        if (buf_len < 8) {
          break;
        }
        // Find newline
        int newlineEnd = -1;
        for (int i = 7; i < buf_len; i++) {
          if (buf[i] == '\n') {
            newlineEnd = i;
            break;
          }
        }

        if (newlineEnd == -1) {
          break;
        }

        // Copy the bytes for the ip and port in their respective variables
        uint32_t ip;
        uint16_t port;
        memcpy(&ip, buf + 1, 4);
        memcpy(&port, buf + 5, 2);

        // Copy the message and addign the newline at the end of the str
        int str_len = newlineEnd - 7;
        char msg[BUF_SIZE];
        memcpy(msg, buf + 7, str_len);
        msg[str_len] = '\0';

        // Binary to string
        char ip_string[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &ip, ip_string, INET_ADDRSTRLEN);

        // Port from network byte order to host with byte order
        uint16_t host = ntohs(port);

        // Message with the ip, host and message
        fprintf(log_fp, "%-15s%-10u%s\n", ip_string, host, msg);
        fflush(log_fp);

        // Move unparsed bytes to the front
        int remaining = buf_len - (newlineEnd + 1);
        if (remaining > 0) {
          memmove(buf, buf + newlineEnd + 1, remaining);
        }
        buf_len = remaining;

        // Server termination
      } else if (msgType == 1) {

        // Find newline for command
        int newlineEnd = -1;
        for (int i = 0; i < buf_len; i++) {
          if (buf[i] == '\n') {
            newlineEnd = i;
            break;
          }
        }

        if (newlineEnd == -1) {
          break;
        }
        fclose(log_fp);
        exit(EXIT_SUCCESS);

        // For garbage data move everything and try to find a valid message
      } else {
        buf_len--;
        memmove(buf, buf + 1, buf_len);
      }
    }
  }
  return NULL;
}

int main(int argCount, char *argMsg[]) {

  // Argument required
  if (argCount != 5) {
    printf("%s <IP> <port number> <# of messages> <log file path> \n",
           argMsg[0]);
    exit(EXIT_FAILURE);
  }

  // Arguments of use by the client
  char *ip_addr = argMsg[1];
  int port_number = atoi(argMsg[2]);
  int messagesCount = atoi(argMsg[3]);
  char *path = argMsg[4];

  struct sockaddr_in addr;
  int sfd;

  sfd = socket(AF_INET, SOCK_STREAM, 0);
  if (sfd == -1) {
    handle_error("socket");
  }

  // Server adress connect
  memset(&addr, 0, sizeof(struct sockaddr_in));
  addr.sin_family = AF_INET;
  addr.sin_port = htons(port_number);

  // Convert IP to binary format for the network
  if (inet_pton(AF_INET, ip_addr, &addr.sin_addr) <= 0) {
    handle_error("inet_pton");
  }

  // connect the server
  int res = connect(sfd, (struct sockaddr *)&addr, sizeof(struct sockaddr_in));
  if (res == -1) {
    handle_error("connect");
  }

  // Log file
  FILE *log_fp = fopen(path, "w");
  if (!log_fp) {
    handle_error("log file");
  }

  // Background listen for the server broadcast (keep the messages in order)
  pthread_t th;
  struct thread_args *args = malloc(sizeof(struct thread_args));
  args->sfd = sfd;
  args->log_fp = log_fp;

  if (pthread_create(&th, NULL, recieve_thread, args) != 0) {
    handle_error("pthread_create");
  }

  // Fuzzer
  for (int i = 0; i < messagesCount; i++) {
    uint8_t bytes[10];
    char hex_string[21];

    getentropy(bytes, sizeof(bytes));

    if (convert(bytes, sizeof(bytes), hex_string, sizeof(hex_string)) != 0) {
      exit(EXIT_FAILURE);
    }

    char out_msg[BUF_SIZE];
    int len = sprintf(out_msg, "%c%s\n", 0, hex_string);

    int sent_bytes = 0;

    while (sent_bytes < len) {
      ssize_t s = write(sfd, out_msg + sent_bytes, len - sent_bytes);
      if (s <= 0) {
        break;
      }
      sent_bytes = sent_bytes + s;
    }
  }

  char type1Msg[2] = {1, '\n'};
  int sent = 0;
  while (sent < 2) {
    ssize_t s = write(sfd, type1Msg + sent, 2 - sent);
    if (s <= 0) {
      break;
    }
    sent = sent + s;
  }

  pthread_join(th, NULL);

  close(sfd);
  exit(EXIT_SUCCESS);
}
