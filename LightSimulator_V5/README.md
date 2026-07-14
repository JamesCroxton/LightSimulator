Commands to setup and run:
Build:
cd LightSimulator_V5
cmake -S . -B build -G Ninja
OR 
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release

cmake --build build

Run:
.\LightSimulator.exe  