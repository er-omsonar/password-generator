KUNAL PROJECT


#include <stdio.h>
#include <string.h>
#include <conio.h>

// Function to calculate checksum
unsigned int compute_checksum(unsigned char *data, int len)
{
    unsigned long sum = 0;
    unsigned int word;
    int i;

    // Add 16-bit words
    for(i = 0; i < len - 1; i += 2)
    {
        word = (data[i] << 8) | data[i + 1];
        sum = sum + word;
    }

    // If odd byte remains
    if(len % 2 != 0)
    {
        word = (data[len - 1] << 8);
        sum = sum + word;
    }

    // Add carry bits
    while(sum >> 16)
    {
        sum = (sum & 0xFFFF) + (sum >> 16);
    }

    // Return 16-bit complement
    return ((unsigned int)(~sum)) & 0xFFFF;
}

void main()
{
    unsigned char packet[] = "Hello TCP Packet";
    unsigned char verify_packet[100];

    unsigned int checksum;
    unsigned int validation;

    int len, i;

    clrscr();

    len = strlen(packet);

    // Initialize verify packet
    for(i = 0; i < 100; i++)
    {
        verify_packet[i] = 0;
    }

    // Generate checksum
    checksum = compute_checksum(packet, len);

    // Copy original packet
    memcpy(verify_packet, packet, len);

    // Attach checksum bytes
    verify_packet[len] =
        (checksum >> 8) & 0xFF;

    verify_packet[len + 1] =
        checksum & 0xFF;

    // Validate checksum
    validation =
        compute_checksum(verify_packet, len + 2);

    printf("===== TCP CHECKSUM SYSTEM =====\n\n");

    printf("Payload            : %s\n",
           packet);

    printf("Generated Checksum : 0x%X\n",
           checksum);

    printf("Validation Check   : 0x%X\n",
           validation);

    // Final result
    if(validation == 0)
        printf("Data Received Successfully\n");
    else
        printf("Error in Data\n");

    printf("\n================================\n");

    getch();
}







---------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------------------


OM PROJECT



#include <stdio.h>
#include <string.h>

#ifdef __TURBOC__
#include <conio.h>
#endif

#define MAX_DEVICES 3
#define CACHE_SIZE 5

// ARP Cache Entry
typedef struct {
    char ip[16];
    char mac[18];
} ArpEntry;

// Network Device
typedef struct {
    char name[20];
    char ip[16];
    char mac[18];

    ArpEntry cache[CACHE_SIZE];
    int count;
} Device;

Device LAN[MAX_DEVICES];

// Add entry to ARP cache
void addToCache(Device *d, char ip[], char mac[]) {

    int i;

    for(i = 0; i < d->count; i++) {

        if(strcmp(d->cache[i].ip, ip) == 0) {

            strcpy(d->cache[i].mac, mac);
            return;
        }
    }

    if(d->count < CACHE_SIZE) {

        strcpy(d->cache[d->count].ip, ip);
        strcpy(d->cache[d->count].mac, mac);

        d->count++;
    }
}

// Print ARP cache
void printCache(Device d) {

    int i;

    printf("\n--- ARP Cache of %s ---\n", d.name);

    for(i = 0; i < d.count; i++) {

        printf("IP : %-15s -> MAC : %s\n",
               d.cache[i].ip,
               d.cache[i].mac);
    }
}

// Send ARP reply
void sendReply(char target[], char senderIP[], char senderMAC[]) {

    int i;

    for(i = 0; i < MAX_DEVICES; i++) {

        if(strcmp(LAN[i].name, target) == 0) {

            printf("[%s] Received ARP Reply : %s is at %s\n",
                   LAN[i].name,
                   senderIP,
                   senderMAC);

            addToCache(&LAN[i], senderIP, senderMAC);
        }
    }
}

// Broadcast ARP request
void arpRequest(Device *sender, char targetIP[]) {

    int i;

    printf("[%s] ARP Request : Who has %s ? Tell %s\n",
           sender->name,
           targetIP,
           sender->ip);

    for(i = 0; i < MAX_DEVICES; i++) {

        // Skip sender
        if(strcmp(LAN[i].ip, sender->ip) == 0)
            continue;

        // Store sender info
        addToCache(&LAN[i], sender->ip, sender->mac);

        // Target found
        if(strcmp(LAN[i].ip, targetIP) == 0) {

            printf(" -> [%s] Target Found! Sending Reply\n",
                   LAN[i].name);

            sendReply(sender->name,
                      LAN[i].ip,
                      LAN[i].mac);
        }
    }
}

int main() {

#ifdef __TURBOC__
    clrscr();
#endif

    // Host A
    strcpy(LAN[0].name, "Host A");
    strcpy(LAN[0].ip, "192.168.1.5");
    strcpy(LAN[0].mac, "AA:BB:CC:DD:EE:01");
    LAN[0].count = 0;

    // Host B
    strcpy(LAN[1].name, "Host B");
    strcpy(LAN[1].ip, "192.168.1.10");
    strcpy(LAN[1].mac, "AA:BB:CC:DD:EE:02");
    LAN[1].count = 0;

    // Router
    strcpy(LAN[2].name, "Router");
    strcpy(LAN[2].ip, "192.168.1.1");
    strcpy(LAN[2].mac, "00:11:22:33:44:55");
    LAN[2].count = 0;

    printf("===== ARP Protocol Simulation =====\n");

    // Host A requests MAC of Host B
    arpRequest(&LAN[0], "192.168.1.10");

    // Print cache
    printCache(LAN[0]);

#ifdef __TURBOC__
    getch();
#endif

    return 0;
}
