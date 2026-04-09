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
        
        logging.info(f"Initiating discovery. Broadcasting to {target}")
        
        try:
            self.sock.sendto(b'DISCOVER', target)
            logging.info("DISCOVER packet sent successfully.")
        except OSError as e:
            logging.error(f"Failed to send broadcast packet: {e}")
            return servers

        logging.info(f"Listening for replies for {self.timeout} seconds...")
        
        while True:
            try:
                data, addr = self.sock.recvfrom(1024)
                logging.debug(f"Received payload: {data} from {addr}")
                
                if data == b'OFFER':
                    servers.add(addr[0])
                    logging.info(f"Valid OFFER registered from {addr[0]}")
            except socket.timeout:
                logging.info("Listening timeout reached. Ending discovery phase.")
                break
            except Exception as e:
                logging.error(f"Client exception occurred: {e}")
                break

        return servers

if __name__ == "__main__":
    client = UDPClient()
    found_servers = client.discover_servers()
    
    logging.info(f"Discovery complete. Total servers found: {len(found_servers)}")
    
    for server_ip in found_servers:
        print(server_ip)
