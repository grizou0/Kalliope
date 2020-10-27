# Kalliope
Interpréteur de commande avec Raspberry Pi4
Configuration Raspberry Pi 4
-----------------------------
1-Installation
bash -c "$(curl -sL https://raw.githubusercontent.com/kalliope-project/kalliope/master/install/rpi_install_kalliope.sh)"

ou en manuel:
git clone https://github.com/kalliope-project/kalliope.git
cd kalliope
sudo python3 setup.py install

Patch français
--------------
git clone https://github.com/kalliope-project/kalliope_starter_fr.git
cd kalliope_starter_fr
kalliope start

Erreur greenlet:
---------------
compatibilite python3.7 et greenlet:
greenlet version 0.4.17 a remplacer par la version 0.4.14

pip install --upgrade greenlet --ignore-installed greenlet
ou on peut spécifier la version à installer.
pip install greenlet==0.4.14

Config Kalliope:
----------------
Creation log:
-------------
sudo mkdir /var/log/kalliope
sudo chown pi: /var/log/kalliope
Installation alsamixer (micro et sortie vers USB)
-------------------------------------------------
sudo apt-get install alsa-utils

sudo usermod -a -G pulse-access pi

sudo nano /etc/systemd/system/pulseaudio.service

[Unit]
Description=PulseAudio system server
After=network.target

[Service]
Type=notify
ExecStart=pulseaudio --daemonize=no --system --realtime --log-target=journal
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

Liste des IO
------------
pactl list sources short

pi@raspberrypi:~/kalliope_starter_fr $ pactl list sources short
0       alsa_output.usb-Burr-Brown_from_TI_USB_audio_CODEC-00.analog-stereo.monitor     module-alsa-card.c      s16le 2ch 48000Hz       IDLE
1       alsa_input.usb-Burr-Brown_from_TI_USB_audio_CODEC-00.analog-mono        module-alsa-card.c      s16le 1ch 48000Hz       IDLE
2       alsa_output.platform-bcm2835_audio.analog-stereo.monitor        module-alsa-card.c      s16le 2ch 44100Hz       IDLE
3       alsa_output.hw_0_0.monitor      module-alsa-sink.c      s16le 2ch 48000Hz       IDLE

Set output
----------
pactl set-default-sink 0

Set input
---------
pactl set-default-source 1

Ajustage micro avec alsamixer (25db micro)
faire apres pour enregistrer les paramètres: sudo alsactl store 

Startup
-------
Au startup, kalliope crée un fichier /etc/systemd/system/kalliope.service.
Remplacer la ligne /home/pi/my_kalliope_config/ par

faire sudo nano /etc/systemd/system/kalliope.service
[Unit]
Description=Kalliope

[Service]
WorkingDirectory=/home/pi/my_kalliope_config/

Environment='STDOUT=/var/log/kalliope.log'
Environment='STDERR=/var/log/kalliope.err.log'
ExecStart=/bin/bash -c "/usr/local/bin/kalliope start --debug > ${STDOUT} 2> ${STDERR}"
User=pi

[Install]
WantedBy=multi-user.target

--------------------------------------------------------------
Activation au startup:
sudo systemctl enable kalliope
sudo systemctl start kalliope
----------------------------------------------------------------------------------------------------------------------------------
fichier Start.sh:
pactl set-default-source 1
pactl set-default-sink 0
kalliope start

Fichier settings.yml
--------------------
Configuration trigger:
trigger: snowboy
  
Fichier brain.yml
-----------------
contient les fichier yml dans le répertoire brains (include)
contient la réponse not found et je suis prêt

global_variables/variable.ym contiennent les variables globales : examples name

répertoire templates > fichier j2 interprétation des variables
systemdate_template.j2
rss_new_le_monde.j2
wikipedia_search.j2

Exemple de brain:
----------------
say.yml
-------
- name: "say-hello-fr"
  signals:
    - order: "Bonjour"
  neurons:
    - say:
        message: "Bonjour {{name}}"
- name: "say-merci"
  signals:
    - order: "merci"
  neurons:
    - say:
        message: 
          - "de rien {{name}}"
          - "le plaisir est pour moi {{name}}"
          - "sans problème {{name}}"

lancement d'un programme 
geany.yml
---------
  - name: "geany"
    signals:
      - order: "Jenny"
      - order: "éditeur"
      - order: "éditeur c"
      - order: "ouvre éditeur c"
      - order: "ouvre éditeur"
      - order: "ouvre l'éditeur"
      - order: "lance l'éditeur"
    neurons:
      - say:   
          message: "ouverture éditeur jenny"
      - shell:
          cmd: "geany"

Désactivation du trigger snowboy(kallioppé)
-------------------------------------------
  - name: "triggeroff"
    signals:
      - order: "ecoute"
      - order: "écoute"
      - order: "trigger off"
    neurons:
      - say:
          message: "triguere off"
      - signals:
          notification: "skip_trigger"
          payload:
              status: "True"
  - name: "triggeron"
    signals:
      - order: "bye"
      - order: "trigger on"
    neurons:
      - say:
          message: "triguere one"
      - signals:
          notification: "skip_trigger"
          payload:
              status: "False"

NeureoTransmetteur (branchement suivant la réponse)
------------------
  - name: "synapse1"
    signals:
      - order: "pose moi une question"
      - order: "pose-moi une question"
    neurons:
      - say:
          message: "aimez vous les frites?"
      - neurotransmitter:
          from_answer_link:
            - synapse: "synapse2"
              answers:
                - "absolument"
                - "peut-être"
                - "oui"
            - synapse: "synapse3"
              answers:
                - "pas du tout"
                - "non"
          default: "synapse4"

  - name: "synapse2"
    signals:
      - order: "synapse2"
    neurons:
      - say:
          message: "Vous aimez les frites! Moi aussi!"


  - name: "synapse3"
    signals:
      - order: "synapse3"
    neurons:
      - say:
          message: "vous n'aimez pas les frites. c'est pas grave."

  - name: "synapse4"
    signals:
      - order: "synapse4"
    neurons:
      - say:
          message: "Je n'ai pas compris votre réponse"
          

Ecriture d'un fichier suivant une commande.
Ce fichier permet à un soft externe d'exécuter des commande suivant l'ordre enregistré 
# execution d'un programme 
#------------------------------------
#  - name: "commande1"
#    signals:
#      - order: "regarde à droite"
#    neurons:
#      - shell:
# lance le soft avec les parametres 
#          cmd: "lxterminal -e ./ecrit 0" "# execution d'un programme 
#------------------------------------
#  - name: "commande1"
#    signals:
#      - order: "regarde à droite"
#    neurons:
#      - shell:
# lance le soft avec les parametres 
#          cmd: "lxterminal -e ./ecrit 0" # exécution du programme avec un parametre "0"
#      - say:
#           message: "soft exécuté"
#----------------------------------
  - name: "commande2"
    signals:
      - order: "fin"
    neurons:
      - shell:
          cmd: "echo 02 > commande.txt" #ecriture d'un fichier "commande.txt" avec 02 
      - say:
           message: "soft exécuté"
......
