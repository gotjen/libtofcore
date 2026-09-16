# LibToFCore

Libraries to interface with the PreAct TOF sensors.

## Environment Setup
Requirements:

| Dependency | Minimum | Tested with |
|---|---|---|
| CMake | 3.16 | 4.2.3 |
| C++ compiler | C++17 support | GCC 15.2 |
| Boost (headers + `program_options`) | 1.70 | 1.90.0 |
| Python + development headers (if building Python bindings) | 3.8 | 3.14 |
| libudev-dev (prerequisite for libusbp) | | |
| libusbp | 1.3 (bundled in `tofcore/third_party` if not installed) | bundled |
| git + network access | | |

pybind11 (v3.0.1) and GoogleTest are downloaded automatically by CMake at configure time.

Debian/Ubuntu package installation
```
sudo apt-get update -y
sudo apt-get install cmake g++ git pkg-config libudev-dev \
    libboost-dev libboost-program-options-dev python3-dev
```

Python setuptools installation (only needed for `make pytofcore`, might need Python venv setup)
```
pip install setuptools
```

USB Udev Rules

Create new udev rules file for usb. Example: `etc/udev/rules.d/99-usb-rules.rules`
```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="35FA", ATTRS{idProduct}=="0D0F", MODE:="0666"
```

## Build and Install

### Build
```
cmake -B build
cmake --build build -j$(nproc)
```
(or `make build`)

Useful configure options:

- `-DBUILD_PYTHON_BINDINGS=OFF` skips the Python module (on by default).
- `-DPython_EXECUTABLE=/usr/bin/python3` selects which Python the bindings are built for. Use it when several are installed, for example with pyenv.

Build outputs:

- `build/tofcore/libtofcore.a`
- the command line tools under `build/tofcore/test/functional-tests/`, for example `tof-stat`
- `build/tofcore/wrappers/python/pytofcore.cpython-*.so`

To try the Python module without installing it:
```
PYTHONPATH=build/tofcore/wrappers/python python3 -c "import pytofcore"
```

### Install to local system
```
cmake --build build -- install  # Installs to /usr/local on UNIX systems

```

## Python Bindings Installation

To install the python package into your personal python site-packages directory:

```
make pytofcore
```

## Testing

### C++ Unit Tests
No camera required:
```
ctest --test-dir build/tofcore --output-on-failure
```
(or `make cpp_unit_test`)

### Python Unit Tests
To run unit tests verifying behavior when no camera is connected, use the following commnad from

project's root directory: 
```
python3 -m pytest -m "not functional and not sdram_selftest" -v .
```

### Functional Tests
Functional tests with a camera connected to PC can be executed with the following commands:

#### For variants without ethernet capabilities
```
python3 -m pytest -m "functional" -k "not test_ip_measurement_endpoint" -v .
```

#### For variants with ethernet
```
python3 -m pytest -m "functional" -v .
```

#### SDRAM self-test
NOTE: connection with the device will be lost, as the device will reset:
```
python3 -m pytest -m "sdram_selftest" -v .
```

#### Optional: Specific URI
For connecting to a specific connected device.

##### Examples

Linux serial type device
```
python3 -m pytest -m "functional" -v . --sensor-uri=tofserial:/dev/ttyACM12?baudrate=115200
```

Windows COM port
```
python3 -m pytest -m "functional" -v . --sensor-uri=tofserial:COM1
```

IP network device
```
python3 -m pytest -m "functional" -v . --sensor-uri=tofnet:10.10.31.180
```
