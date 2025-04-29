#!/usr/bin/env python
import os
import sys
# from scons_compiledb import compile_db # Import the compile_db function # Call the compile_db function to enable compile_commands.json generation 
# compile_db()


# Import the SConstruct from godot-cpp
env = SConscript("godot-cpp/SConstruct")

# Add necessary include directories
env.Append(CPPPATH=[
    "src/glm/",
    "src/utility/",
    "src/",
    "src/sdf/",
    "src/voxel_terrain/",
    "src/voxel_terrain/meshing",
    "src/voxel_terrain/meshing/adaptive_surface_nets",
    "src/voxel_terrain/world",
    "src/voxel_terrain/population",
    "src/voxel_terrain/population/details",
    "src/voxel_terrain/population/features",
])

# # Add main source files
sources = Glob("src/*.cpp") + Glob("src/utility/*.cpp") + Glob("src/sdf/*.cpp") + \
      Glob("src/voxel_terrain/*.cpp") + Glob("src/voxel_terrain/meshing/*.cpp") + \
        Glob("src/voxel_terrain/meshing/adaptive_surface_nets/*.cpp") + \
            Glob("src/voxel_terrain/meshing/stitched_surface_nets/*.cpp") +\
            Glob("src/voxel_terrain/population/*.cpp") + Glob("src/voxel_terrain/population/details/*.cpp") + \
            Glob("src/voxel_terrain/population/features/*.cpp")

#compiler flags
if env['PLATFORM'] == 'windows':
    if env['CXX'] == 'x86_64-w64-mingw32-g++':
        env.Append(CXXFLAGS=['-std=c++11'])  # Example flags for MinGW
    elif env['CXX'] == 'cl':
        env.Append(CXXFLAGS=['/EHsc'])  # Apply /EHsc for MSVC

# Handle different platforms
if env["platform"] == "macos":
    library = env.SharedLibrary(
        "project/addons/jar_voxel_terrain/bin/jar_voxel_terrain.{}.{}.framework/jar_voxel_terrain.{}.{}".format(
            env["platform"], env["target"], env["platform"], env["target"]
        ),
        source=sources,
    )
elif env["platform"] == "ios":
    if env["ios_simulator"]:
        library = env.StaticLibrary(
            "project/addons/jar_voxel_terrain/bin/jar_voxel_terrain.{}.{}.simulator.a".format(env["platform"], env["target"]),
            source=sources,
        )
    else:
        library = env.StaticLibrary(
            "project/addons/jar_voxel_terrain/bin/jar_voxel_terrain.{}.{}.a".format(env["platform"], env["target"]),
            source=sources,
        )
else:
    library = env.SharedLibrary(
        "project/addons/jar_voxel_terrain/bin/jar_voxel_terrain{}{}".format(env["suffix"], env["SHLIBSUFFIX"]),
        source=sources,
    )

Default(library)

# --- Linux Only: Auto-Copy TBB runtime if needed ---
if env["platform"] == "linux":
    env.Append(LIBS=["tbb"])
    # Check if TBB runtime is present
    # This is a workaround for the fact that SCons doesn't automatically copy shared libraries
    # to the target directory. We need to manually copy the TBB runtime if it's not already there.
    import shutil

    # Try multiple common locations
    tbb_possible_paths = [
        "/usr/lib/libtbb.so.12",
        "/usr/lib64/libtbb.so.12",
        "/lib/libtbb.so.12",
        "/lib64/libtbb.so.12",
    ]

    tbb_source = None
    for path in tbb_possible_paths:
        if os.path.exists(path):
            tbb_source = path
            break

    tbb_target = "project/addons/jar_voxel_terrain/bin/libtbb.so.12"

    if tbb_source:
        if not os.path.exists(tbb_target):
            print(f"✅ Found TBB at {tbb_source}. Copying to {tbb_target}...")
            os.makedirs(os.path.dirname(tbb_target), exist_ok=True)
            shutil.copy(tbb_source, tbb_target)
        else:
            print(f"✅ TBB runtime already present: {tbb_target}")
    else:
        print("\n❌ ERROR: Intel TBB runtime (libtbb.so.12) not found on your system!\n")
        print("Please install TBB before building GDVoxelTerrain:")
        print("    Arch Linux:    sudo pacman -S tbb")
        print("    Ubuntu/Debian: sudo apt install libtbb-dev")
        print("    Fedora:        sudo dnf install tbb-devel")
        print("    OpenSUSE:      sudo zypper install tbb-devel\n")
        print("After installing, rebuild the project using: scons platform=linux\n")
        Exit(1)
