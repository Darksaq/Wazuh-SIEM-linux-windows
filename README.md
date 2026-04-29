
    
# 1. Instalacja Wazuh Manager na Linux (Parrot OS)
Najpierw przygotowujemy system, dodajemy klucze i uruchamiamy asystenta instalacji, który skonfiguruje całość (Manager, Indexer, Dashboard).

```sudo apt update && sudo apt full-upgrade -y```

## - Dodanie klucza GPG
```curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg```

## - Pobranie i uruchomienie skryptu instalacyjnego
`curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i`

Po zakończeniu instalacji w terminalu wyświetli się login i hasło.

# 2. Weryfikacja statusu usług
Sprawdzamy, czy wszystko działa:

`sudo systemctl status wazuh-manager`

`sudo systemctl status wazuh-indexer`

`sudo systemctl status wazuh-dashboard`

# 3. Dostęp do Dashboardu Web
Aby zalogować się do panelu graficznego, sprawdź adres IP maszyny i wpisz go w przeglądarce.

## - Sprawdzenie lokalnego adresu IP
`ifconfig`

Adres URL:    https://<twoje_ip> 

(login i hasło z termianala po instalacji).

# 4. Rejestracja Agenta na Managerze (Generowanie klucza)
Zanim zainstalujemy agenta na Windowsie, musimy go zarejestrować w systemie Managera, aby uzyskać klucz uwierzytelniający.

`sudo /var/ossec/bin/manage_agents`

#### Kolejne kroki w menu:

Wybierz A, aby dodać agenta.

Nadaj nazwę (np. WindowsHost).

Adres IP pozostaw pusty (chyba że masz statyczne przypisanie).

Po utworzeniu wybierz E, aby wyodrębnić (extract) klucz.

Skopiuj wygenerowany ciąg znaków.

# 5. Instalacja i konfiguracja Agenta na Windows
#### Wygenerować komendę w Web GUI.

Wazuh > Agents > Deploy new agent.

Wypełnij pola zgodnie z Twoim setupem:

Package selection: Wybierz ikonkę Windows.

Wazuh server address: Wpisz lokalny adres IP swojej maszyny z wazuh-manager.

Agent name: Nazwij go np. Windows11.

Group: Możesz zostawić default.

#### Na dole strony pojawi się gotowa komenda PowerShell.

Zaloguj się na maszyne agenta w tym przypadku win11.

Otwórz PowerShell jako Administrator.

Wklej komendę i Enter.

##### Komenda będzie wyglądać mniej więcej tak:

`Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.msi -OutFile ${env:tmp}\wazuh-agent.msi; msiexec.exe /i ${env:tmp}\wazuh-agent.msi /q WAZUH_MANAGER='1.1.1.1' WAZUH_AGENT_NAME='Windows11'`

## Uruchomienie usługi:
Jeśli wolisz terminal, użyj jednej z poniższych komend (jako administrator):

CMD:
`NET START WazuhSvc`

PowerShell:
`Start-Service wazuhsvc`

## Lab
Wazuh-manager: parrotOS

Wazuh-agent: Windows11

W tym momencie Twój agent Windows powinien pojawić się w Dashboardzie jako Active.
