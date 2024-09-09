<p align="center"><img src="./img/rtcRPi.png" width="300"   alt=" " /></p>
<h1 align="center">RTC DS3231 module in a Raspberry Pi </h1> 
<h4 align="right">Agu 24</h4>

<img src="https://img.shields.io/badge/Hardware-Raspberry%20ver%204-red">
<img src="https://img.shields.io/badge/Hardware-Raspberry%203B%2B-red">
<img src="https://img.shields.io/badge/Hardware-Raspberry%20Zero-red">


<br>
Aquí te dejo un script en Python3 para usar el módulo RTC DS3231 en una Raspberry Pi.
El script toma la fecha y hora del RTC al iniciar y la actualiza desde Internet si está disponible.

<p align="center"><img src="./img/rtcRPi1.png" width="300"   alt=" " /></p>
<p align="center"><img src="./img/rtcRPi2.png" width="300"   alt=" " /></p>

<br>

https://www.raspipc.es/blog/anadir-un-reloj-rtc-a-la-raspberry-pi/
https://www.arduinoecia.com.br/como-usar-rtc-ds3231-raspberry-pi/

chrome-extension://laameccjpleogmfhilmffpdbiibgbekf/suspended.html?title=Muchos%20aparatos%20raros%3A%20Instalar%20y%20configurar%20reloj%20RTC%20DS3231%20en%20Raspberry%20Pi%20con%20Raspbian%20Jessie&url=https%3A%2F%2Fmuchosaparatosraros.blogspot.com%2F2016%2F05%2Finstalar-y-configurar-reloj-rtc-ds3231.html&time=1725858673760

https://monkiki.github.io/2017/03/22/rtc-ds3231-en-raspberry-pi.html

```
sudo apt-get update && sudo apt-get full-upgrade
sudo reboot
sudo apt-get install i2c-tools
sudo i2cdetect -y 1
```

```
2. Habilitar I2C en Raspberry Pi:
Ejecuta sudo raspi-config.
Selecciona Interfacing Options y luego selecciona I2C.
Habilita el I2C.
Reinicia la Raspberry Pi con sudo reboot.

3. Instalar herramientas de I2C:
Para verificar que el módulo RTC esté correctamente conectado, instala las herramientas de I2C:

bash
Copiar código
sudo apt-get install i2c-tools
4. Verificar el RTC DS3231:
Ejecuta el siguiente comando para asegurarte de que la Raspberry Pi detecta el RTC:

bash
Copiar código
sudo i2cdetect -y 1
Debes ver el dispositivo en la dirección 0x68. Si aparece, el RTC está correctamente conectado.

5. Configurar el RTC DS3231:
Ahora, vamos a indicarle al sistema que use el RTC DS3231. Para hacer esto:

Abre el archivo config.txt:

bash
Copiar código
sudo nano /boot/config.txt
Agrega la siguiente línea al final del archivo para habilitar el uso del RTC:

bash
Copiar código
dtoverlay=i2c-rtc,ds3231
Guarda y cierra el archivo.

6. Desactivar el reloj por software:
La Raspberry Pi usa un reloj de software por defecto (fake-hwclock). Debemos desactivarlo:

bash
Copiar código
sudo apt-get -y remove fake-hwclock
sudo update-rc.d -f fake-hwclock remove
sudo systemctl disable fake-hwclock
7. Actualizar los scripts de inicio:
El RTC ahora debe configurarse para sincronizar la hora al arrancar. Modifica el archivo /lib/udev/hwclock-set:

bash
Copiar código
sudo nano /lib/udev/hwclock-set
Busca las siguientes líneas y coméntalas agregando # al inicio de cada línea:

bash
Copiar código
if [ -e /run/systemd/system ] ; then
    exit 0
fi
Estas líneas deben quedar así:

bash
Copiar código
#if [ -e /run/systemd/system ] ; then
#    exit 0
#fi
Guarda y cierra el archivo.

8. Reiniciar la Raspberry Pi:
Reinicia la Raspberry Pi para aplicar los cambios:

bash
Copiar código
sudo reboot
9. Verificar la hora del RTC:
Después del reinicio, puedes verificar la hora del RTC ejecutando:

bash
Copiar código
sudo hwclock -r
Si el comando muestra la hora correcta, el RTC está configurado correctamente.

10. Sincronizar el RTC con la hora del sistema:
Si necesitas sincronizar la hora del sistema con el RTC, puedes hacerlo con el siguiente comando:

bash
Copiar código
sudo hwclock -w
Esto grabará la hora actual del sistema en el RTC. Ahora, incluso si la Raspberry Pi se apaga, el RTC mantendrá la hora correcta.

11. Configurar RTC como servicio en SYSTEMD (opcional):
Para que la sincronización del RTC se ejecute como un servicio de sistema en SYSTEMD, podemos crear un archivo de servicio personalizado.

Crea el archivo de servicio:

bash
Copiar código
sudo nano /etc/systemd/system/rtc-sync.service
Agrega el siguiente contenido:

ini
Copiar código
[Unit]
Description=Sincronización de tiempo con el RTC
After=network.target

[Service]
ExecStart=/sbin/hwclock --hctosys

[Install]
WantedBy=multi-user.target
Guarda y cierra el archivo, luego habilítalo:

bash
Copiar código
sudo systemctl enable rtc-sync.service
sudo systemctl start rtc-sync.service
Con esto, cada vez que la Raspberry Pi se reinicie, se sincronizará la hora del sistema con el RTC DS3231 automáticamente.
```

<br>


# Service

rtc-sync.service

```
sudo nano /etc/systemd/system/RTC-rpi.service
```
sample: 
sudo systemctl daemon-reload
sudo systemctl restart rtc-sync.service
sudo journalctl -f -u rtc-sync.service
sudo systemctl status rtc-sync.service
sudo journalctl -f -u rtc-sync.service
sudo systemctl disable rtc-sync.service


```bash
[Unit]
Description=RTC-rpi
After=multi-user.target

[Service]
ExecStart=/usr/bin/python3 /home/carjavi/RTC-rpi.py
StandardOutput=append:/home/carjavi/RTC-rpi.log
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
[Unit]
Description=RTC Raspberry Pi Service
After=network.target

[Service]
# Si usas un entorno virtual:
WorkingDirectory=/home/carjavi/
ExecStart=/bin/bash -c 'source /home/carjavi/env/bin/activate && python3 /home/carjavi/RTC-rpi.py'
Restart=always
User=carjavi
Environment=PYTHONUNBUFFERED=1

# Hacer que el servicio inicie después de montar el sistema de archivos (opcional)
ExecStartPre=/bin/sleep 10

[Install]
WantedBy=multi-user.target
```

```bash
[Unit]
Description=RTC-rpi
After=network.target

[Service]
WorkingDirectory=/home/carjavi/
ExecStart=/bin/bash -c 'python3 -u /home/carjavi/RTC-rpi.py'
Restart=on-failure
StandardOutput=append:/home/carjavi/RTC-rpi.log

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl enable RTC-rpi
sudo systemctl start RTC-rpi
sudo reboot
```
```
sudo journalctl -f -u RTC-rpi
sudo systemctl status RTC-rpi
sudo systemctl restart RTC-rpi
sudo systemctl disable RTC-rpi
sudo systemctl stop RTC-rpi
```

<br>

---
Copyright &copy; 2022 [carjavi](https://github.com/carjavi). <br>
```www.instintodigital.net``` <br>
carjavi@hotmail.com <br>
<p align="center">
    <a href="https://instintodigital.net/" target="_blank"><img src="./img/developer.png" height="100" alt="www.instintodigital.net"></a>
</p>


# RTC-rpi
Install an RTC DS3231 module in a Raspberry Pi
