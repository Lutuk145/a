import socket
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

class UDPClient:
    def __init__(self, port=2222, timeout=3.0, broadcast_ip='172.31.255.255'):
        self.port = port
        self.timeout = timeout
        self.broadcast_ip = broadcast_ip
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
        self.sock.settimeout(self.timeout)

    def discover_servers(self):
        servers = set()
        target = (self.broadcast_ip, self.port)
        
        try:
            self.sock.sendto(b'DISCOVER', target)
        except OSError as e:
            logging.error(f"Send failed: {e}")
            return servers

        while True:
            try:
                data, addr = self.sock.recvfrom(1024)
                if data == b'OFFER':
                    servers.add(addr[0])
            except socket.timeout:
                break
            except Exception as e:
                logging.error(f"Receive error: {e}")
                break

        return servers

if __name__ == "__main__":
    client = UDPClient()
    found = client.discover_servers()
    
    for ip in found:
        print(ip)
