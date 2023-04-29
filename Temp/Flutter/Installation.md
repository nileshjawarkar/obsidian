
1) [Download SDK](https://docs.flutter.dev/get-started/install)
2) Unzip and add bin folder to path
3) run - flutter doctor
4) install other dependencies


Linux specific issue -
error :
```
/usr/bin/ld: cannot find -lstdc++: No such file or directory
```


Install following lib to fix the issue -
```
sudo apt install libstdc++-12-dev
```