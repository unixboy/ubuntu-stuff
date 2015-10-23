
Logitech has a Unifying receiver::

  sudo add-apt-repository ppa:daniel.pavel/solaar   
  sudo apt-get update   
  sudo apt-get install solaar   



SSH stuff::

  cat ~/.ssh/id_rsa.pub | ssh USER@HOST "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
