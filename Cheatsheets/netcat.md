#server
nc -l port

# client
echo hello | nc source port

# Send files over the network with

## receiver

netcat -l ip port | sudo tar xf -

the hyphen reprensents sdin

## sender

sudo tar --create --file - path_to_directory | nc ip port
the hypens represents sdout (send compressed files to )
