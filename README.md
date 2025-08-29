# Shooter

I also had a few scripts in the same folder as the linux binary (Same folder as where FPSTest.sh is stored) to try change settings.

Run the game with ./run.sh

## run.sh

```
#!/bin/bash
#export LSFG_DLL_PATH="$HOME/.var/app/com.valvesoftware.Steam/.steam/debian-installation/steamapps/common/Lossless Scaling/Lossless.dll"
#export ENABLE_LSFG=0
#export LSFG_MULTIPLIER=2
#export LSFG_LOG_FILE=lsfg.log

#./FPSTest.sh -ExecCmds="sg.ResolutionQuality 25; sg.ViewDistanceQuality 0; sg.AntiAliasingQuality 0; sg.ShadowQuality 0; sg.PostProcessQuality 0; sg.TextureQuality 1; sg.EffectsQuality 0; sg.FoliageQuality 0; r.Lumen.Reflections 0; r.Lumen.GlobalIllumination 0; r.Shadow.Virtual.Enable 0; r.BloomQuality 0; r.MotionBlurQuality 0; r.AmbientOcclusionLevels 0; r.DepthOfFieldQuality 0; r.ScreenPercentage 50; stat unit"

#./FPSTest.sh -ExecCmds="sg.ResolutionQuality 25; r.ScreenPercentage 50"

#./FPSTest.sh -ExecCmds="r.Shadow.Virtual.Enable 0; r.Shadow.CSM.MaxCascades 1; r.Shadow.MaxResolution 512; r.ShadowQuality 0; toggleconsole; stat GPU; stat unit"


# Somehow clouds in UE can us as much as 2MS of render time on a 3050. So FPS may decerase looking at the sky.

#./FPSTest.sh -ExecCmds="r.Shadow.Virtual.Enable 1; r.Shadow.CSM.MaxCascades 1; r.Shadow.MaxResolution 512; r.ShadowQuality 0; r.VolumetricCloud.MaxSamples 16; r.VolumetricCloud.ShadowQuality 0; r.VolumetricCloud.EnableShadows 0; r.VolumetricCloud.LODFactor 4; r.Lumen.Reflections 0; r.Lumen.GlobalIllumination 0; r.Lumen.DiffuseIndirect 0; r.RayTracing.Reflections 0" -statfile

./launcher.sh
```

## launcher.sh

```
#!/bin/bash

CONFIG_DIR="./FPSTest/Saved/Config/Linux"
ENGINE_INI="$CONFIG_DIR/Engine.ini"
GAMEUSER_INI="$CONFIG_DIR/Engine.GameUserSettings.ini"
PRESET_FILE="$CONFIG_DIR/preset.cfg"

# Ensure config dir exists
mkdir -p "$CONFIG_DIR"

# Load saved preset if available
if [ -f "$PRESET_FILE" ]; then
    source "$PRESET_FILE"
else
    PRESET_NAME="Performance"
    TSR=0
    AA_METHOD=0      # 0=None, 1=FXAA
    FRAME_GEN=0
    RES_QUALITY=100
    SCREEN_PERCENT=100
fi

# --- Menu ---
echo "Select preset:"
echo "1) Performance"
echo "2) Quality"
echo "3) High Quality"
echo "4) Custom"
read -p "> " preset_choice

case "$preset_choice" in
    1)
      	PRESET_NAME="Performance"
        TSR=0
	AA_METHOD=0
        FRAME_GEN=0
        RES_QUALITY=50
        SCREEN_PERCENT=50
        ;;
    2)
      	PRESET_NAME="Quality"
        TSR=0
	AA_METHOD=1
        FRAME_GEN=0
        RES_QUALITY=75
        SCREEN_PERCENT=75
        ;;
    3)
      	PRESET_NAME="High Quality"
        TSR=0
	AA_METHOD=2
        FRAME_GEN=0
        RES_QUALITY=100
        SCREEN_PERCENT=100
        ;;
    4)
      	echo "Custom settings:"
        read -p "Enable TSR? (0=off, 1=on) [$TSR]: " input; [[ $input ]] && TSR=$input
        read -p "AA method (0=None, 1=FXAA, 2=TAA) [$AA_METHOD]: " input; [[ $input ]] && AA_METHOD=$input
        read -p "Enable Frame Generation? (0=off, 1=on) [$FRAME_GEN]: " input; [[ $input ]] && FRAME_GEN=$input
        read -p "Resolution Quality % (25-100) [$RES_QUALITY]: " input; [[ $input ]] && RES_QUALITY=$input
        read -p "Screen Percentage % (25-100) [$SCREEN_PERCENT]: " input; [[ $input ]] && SCREEN_PERCENT=$input
        ;;
    *)
      	echo "Invalid selection, using previous preset: $PRESET_NAME"
        ;;
esac

# Save preset
cat > "$PRESET_FILE" <<EOL
PRESET_NAME="$PRESET_NAME"
TSR=$TSR
AA_METHOD=$AA_METHOD
FRAME_GEN=$FRAME_GEN
RES_QUALITY=$RES_QUALITY
SCREEN_PERCENT=$SCREEN_PERCENT
EOL

# --- Backup existing INIs ---
[ -f "$ENGINE_INI" ] && cp "$ENGINE_INI" "$ENGINE_INI.bak"
[ -f "$GAMEUSER_INI" ] && cp "$GAMEUSER_INI" "$GAMEUSER_INI.bak"

# --- Write Engine.ini ---
cat > "$ENGINE_INI" <<EOL
[/Script/Engine.RendererSettings]
# Shadows
r.Shadow.Virtual.Enable=1
r.Shadow.CSM.MaxCascades=1
r.Shadow.MaxResolution=256
r.ShadowQuality=0

# Volumetric Clouds
r.VolumetricCloud.MaxSamples=16
r.VolumetricCloud.ShadowQuality=0
r.VolumetricCloud.EnableShadows=0
r.VolumetricCloud.LODFactor=4

# Post-processing
r.MotionBlurQuality=0
r.BloomQuality=0
r.AmbientOcclusionLevels=0
r.DepthOfFieldQuality=0
r.RefractionQuality=0

# Effects
r.LensFlareQuality=0
r.Fog=0
r.StationaryLightShadowQuality=0
r.DirectionalLightDistanceField=0

# Disable Lumen (Linux build doesn't support)
r.Lumen.GlobalIllumination=0
r.Lumen.Reflections=0
r.Lumen.DiffuseIndirect=0
r.Lumen.ScreenProbeGather=0

# Lighting
r.DynamicGlobalIlluminationMethod=0
r.ReflectionsMethod=0
r.ReflectionEnvironment=1
r.StaticLightingLevelScale=1.0
r.NumIndirectLightingBounces=1
r.SkyLightIntensity=1.0

# Resolution / Scaling
sg.ResolutionQuality=$RES_QUALITY
r.ScreenPercentage=$SCREEN_PERCENT

# Anti-Aliasing / TSR
r.AntiAliasingMethod=$([[ $TSR -eq 1 ]] && echo 2 || echo $AA_METHOD)
r.TSR.Enabled=$TSR
r.TSR.Quality=0
r.TemporalAA.Upsampling=$TSR

[/Script/Engine.InputSettings]
ConsoleKey=Tilde

[/Script/Engine.GameEngine]
bEnableConsole=True
EOL

# --- Write GameUserSettings.ini ---
cat > "$GAMEUSER_INI" <<EOL
#[/Script/Engine.GameUserSettings]
bUseVSync=False
PreferredFullscreenMode=0
bUseDynamicResolution=True
r.AntiAliasingMethod=$([[ $TSR -eq 1 ]] && echo 2 || echo $AA_METHOD)
EOL

# --- Frame Generation Environment ---
export LSFG_DLL_PATH="$HOME/.var/app/com.valvesoftware.Steam/.steam/debian-installation/steamapps/common/Lossless Scaling/Lossless.dll"
export ENABLE_LSFG=$FRAME_GEN
export LSFG_MULTIPLIER=2
export LSFG_LOG_FILE=lsfg.log
```

# --- Launch the game ---
./FPSTest.sh -statfile`

