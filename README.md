import subprocess
import sys
import os

def process_audio(wav_file, model_name="base.en"):
    model = f"./models/ggml-{model_name}.bin"
    
    # For the newer whisper.cpp builds on Windows
    exe_path = "./build/bin/whisper-cli.exe"
    
    if not os.path.exists(exe_path):
        raise FileNotFoundError(
            f"Whisper executable not found at: {exe_path}\n"
            f"Please ensure whisper.cpp is built correctly."
        )
    
    # Check if the model file exists
    if not os.path.exists(model):
        raise FileNotFoundError(
            f"Model file not found: {model}\n\n"
            f"Download a model with this command:\n\n"
            f"> bash ./models/download-ggml-model.sh {model_name}\n\n"
        )
    
    if not os.path.exists(wav_file):
        raise FileNotFoundError(f"WAV file not found: {wav_file}")
    
    # Use list format for better path handling (especially with spaces)
    command = [exe_path, "-m", model, "-f", wav_file, "-nt"]
    
    # Execute the command
    process = subprocess.Popen(
        command,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        shell=False
    )
    
    output, error = process.communicate()
    
    if process.returncode != 0 and error:
        raise Exception(f"Error processing audio: {error.decode('utf-8')}")
    
    decoded_str = output.decode("utf-8").strip()
    processed_str = decoded_str.replace("[BLANK_AUDIO]", "").strip()
    
    return processed_str

def main():
    if len(sys.argv) >= 2:
        wav_file = sys.argv[1]
        model_name = sys.argv[2] if len(sys.argv) == 3 else "base.en"
        try:
            result = process_audio(wav_file, model_name)
            print(result)
        except Exception as e:
            print(f"Error: {e}", file=sys.stderr)
    else:
        print("Usage: python whisper_processor.py <wav_file> [<model_name>]")

if __name__ == "__main__":
    main()
