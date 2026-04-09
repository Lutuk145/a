import socket

def main():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
    sock.settimeout(3.0)
    
    sock.sendto(b'DISCOVER', ('255.255.255.255', 2222))
    
    servers = set()
    
    while True:
        try:
            data, addr = sock.recvfrom(1024)
            if data == b'OFFER':
                servers.add(addr[0])
        except socket.timeout:
            break
            
    for server_ip in servers:
        print(server_ip)

if __name__ == "__main__":
    main()
