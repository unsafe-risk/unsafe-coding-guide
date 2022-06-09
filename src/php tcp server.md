# PHP TCP server

```php
<?php

$server = stream_socket_server("tcp://localhost:8000", $errno, $errstr);

if ($server === false) {
    throw new Exception("Could not create server: $errstr ($errno)");
}

for (;;) {
    $client = stream_socket_accept($server);
    if ($client === false) {
        echo "Could not accept client\n";
        continue;
    }

    $request = '';
    while (true) {
        $buf = fread($client, 8192);
        $request .= $buf;
        if (strlen($buf) < 8192) {
            break;
        }
    }

    $response = '';
    $response .= $request;

    $request_data = json_decode($request, true);
    $response_data = array();
    $response_data['name'] = $request_data['name'];
    $response_data['greeting'] = "Hello, {$request_data['name']}!";

    if (fwrite($client, json_encode($response_data)) === false) {
        echo "Could not write response\n";
    }

    fclose($client);
}

?>
```

#PHP #TCP #server