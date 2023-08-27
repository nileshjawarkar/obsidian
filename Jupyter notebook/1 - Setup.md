
[Installation link](https://jupyter.org/install)

[kernel supported](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)

# Default -
## 1) Start the server
``` sh
jupyter lab

# Output

[C 2023-08-19 12:36:48.673 ServerApp] 
    
To access the server, open this file in a browser:
        file:///home/vmuser/.local/share/jupyter/runtime/jpserver-1324-open.html
Or copy and paste one of these URLs:
        http://fedsrv01:8888/lab?token=e7404b8d084e6e3fcf97bd7e7248608a36ee8b6eaa13635b
        http://127.0.0.1:8888/lab?token=e7404b8d084e6e3fcf97bd7e7248608a36ee8b6eaa13635b

```

This command will start the jupyter server and provide links to access it.
## 2) User it over network

On another machine, we can not access jupyter server, using above links by replacing the server IP, we need to setup ssh tunnelling
``` sh
ssh -L 8888:localhost:8888 <USER>@<IP>

# Example
ssh -L 8888:localhost:8888 vmuser@192.168.122.53
```

Above command will allow access to the jupyter server installed on IP "192.168.122.53".
Now we can use url generated in step 1.

## 3) Use it in browser

Now you can open link "http://localhost:8888/lab?token=e7404b8d084e6e3fcf97bd7e7248608a36ee8b6eaa13635b" using browser.

# Explicit way -

## 1) Start server using specific IP and PORT
``` sh
jupyter lab --ip 192.168.122.53 --port 8888

# output

W 2023-08-19 22:36:53.434 ServerApp] No web browser found: Error('could not locate runnable browser').
[C 2023-08-19 22:36:53.434 ServerApp] 
    
To access the server, open this file in a browser:
    file:///home/vmuser/.local/share/jupyter/runtime/jpserver-964-open.html
Or copy and paste one of these URLs:
    http://192.168.122.53:8888/lab?token=d2057f9229aafd77640140049bf0fd72667016ca8267ab5e
    http://127.0.0.1:8888/lab?token=d2057f9229aafd77640140049bf0fd72667016ca8267ab5e

```

## 2) Use it in browser
Now you can open link "http://192.168.122.53:8888/lab?token=d2057f9229aafd77640140049bf0fd72667016ca8267ab5e" using browser.

