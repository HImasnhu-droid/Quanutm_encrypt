hey so i have been trying to create an app in android studio i have asked you about it many times but your servers are alwas busy so lets get started so let me give you details of what we  have done so far so my build.gradle(project level ) looks like // build.gradle (Project-level)
buildscript {
    repositories {
        google()
        mavenCentral()
        maven { url "https://chaquo.com/maven" }  // Add Chaquopy repository
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:8.8.0'  // Android Gradle plugin
        classpath "com.chaquo.python:gradle:15.0.1"       // Chaquopy Gradle plugin
    }
}

plugins {
    id("com.android.application") version '8.8.0' apply false
    id("org.jetbrains.kotlin.android") version "1.9.0" apply false
}                   and my build.gradle(app) looks likeplugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("com.chaquo.python")  // Correct
}

android {
    namespace = "com.example.quantumencryptionapp"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.quantumencryptionapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        // Python Configuration
        python {
            version = "3.12"
            pip {
                install "qiskit==0.44.1"
                install "numpy==1.24.4"
            }
        }

        ndk {
            abiFilters  "arm64-v8a", "x86_64"
        }
    }

    // Add this to specify Python source directory
    sourceSets {
        getByName("main") {
            python.srcDir("src/main/python")
        }
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.3"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation platform('androidx.compose:compose-bom:2023.10.01')  // Updated BOM
    implementation 'androidx.compose.ui:ui'
    implementation 'androidx.compose.material:material'
    implementation 'androidx.compose.ui:ui-tooling-preview'
    implementation 'androidx.activity:activity-compose:1.7.2'
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation "com.chaquo.python:kotlin:15.0.1"  // Match Chaquopy version

    debugImplementation 'androidx.compose.ui:ui-tooling'
}                                           and my seetings.gradle looks like // settings.gradle (Project-level)
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
        maven { url "https://chaquo.com/maven" }  // For Chaquopy plugin
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url "https://chaquo.com/maven" }  // For Chaquopy dependencies
    }
}

rootProject.name = "QuantumEncryptionApp"
include(":app") and my file named quanutm_encryption.py which is inside app>src>main>python has following content from qiskit import QuantumCircuit
from qiskit.primitives import Sampler
import random

def string_to_binary(message):
    """Convert a string to its binary representation."""
    return ''.join(format(ord(char), '08b') for char in message)

def binary_to_string(binary_message):
    """Convert a binary representation back to a string."""
    chars = [binary_message[i:i+8] for i in range(0, len(binary_message), 8)]
    return ''.join(chr(int(char, 2)) for char in chars)

def quantum_encrypt_chunk(binary_chunk):
    """Encrypt a small chunk of binary data."""
    num_qubits = len(binary_chunk)
    qc = QuantumCircuit(num_qubits, num_qubits)

    # Encode the binary chunk into the quantum circuit
    for i, bit in enumerate(binary_chunk):
        if bit == '1':
            qc.x(i)

    # Generate a random key
    random_key = [random.choice([0, 1]) for _ in range(num_qubits)]

    # Apply the random key (Z-gates based on the key)
    for i, key in enumerate(random_key):
        if key == 1:
            qc.z(i)

    # Measure the qubits
    qc.measure(range(num_qubits), range(num_qubits))

    # Use the Sampler primitive
    sampler = Sampler()
    result = sampler.run(circuits=[qc]).result()

    # Get the probabilities of measurement outcomes
    counts = result.quasi_dists[0]

    # Find the most probable result as the encrypted binary chunk
    encrypted_binary_chunk = max(counts, key=counts.get)

    # Convert the integer result back to a binary string, preserving the chunk size
    encrypted_binary_chunk_str = format(encrypted_binary_chunk, f'0{num_qubits}b')

    return encrypted_binary_chunk_str, random_key

def quantum_decrypt_chunk(encrypted_binary_chunk, random_key):
    """Decrypt a small chunk of binary data."""
    num_qubits = len(encrypted_binary_chunk)
    qc = QuantumCircuit(num_qubits, num_qubits)

    # Apply the random key (inverse Z-gates based on the key)
    for i, key in enumerate(random_key):
        if key == 1:
            qc.z(i)

    # Encode the encrypted binary chunk into the quantum circuit
    for i, bit in enumerate(encrypted_binary_chunk):
        if bit == '1':
            qc.x(i)

    # Measure the qubits
    qc.measure(range(num_qubits), range(num_qubits))

    # Use the Sampler primitive
    sampler = Sampler()
    result = sampler.run(circuits=[qc]).result()

    # Get the probabilities of measurement outcomes
    counts = result.quasi_dists[0]

    # Find the most probable result as the decrypted binary chunk
    decrypted_binary_chunk = max(counts, key=counts.get)

    # Convert the integer result back to a binary string, preserving the chunk size
    decrypted_binary_chunk_str = format(decrypted_binary_chunk, f'0{num_qubits}b')

    return decrypted_binary_chunk_str

def quantum_decrypt(encrypted_message, random_keys):
    """Decrypt the full encrypted message."""
    encrypted_binary_message = string_to_binary(encrypted_message)

    chunk_size = 8  # Process in chunks of 8 qubits
    decrypted_chunks = []

    # Decrypt each chunk
    for i in range(0, len(encrypted_binary_message), chunk_size):
        encrypted_binary_chunk = encrypted_binary_message[i:i + chunk_size]
        random_key = random_keys[i // chunk_size]  # Get corresponding random key for each chunk
        decrypted_chunk = quantum_decrypt_chunk(encrypted_binary_chunk, random_key)
        decrypted_chunks.append(decrypted_chunk)

    # Combine the decrypted chunks
    decrypted_binary_message = ''.join(decrypted_chunks)
    decrypted_message = binary_to_string(decrypted_binary_message)

    return decrypted_message

def quantum_encrypt(message):
    # Convert the string message to binary
    binary_message = string_to_binary(message)

    chunk_size = 8  # Process in chunks of 8 qubits
    encrypted_chunks = []
    random_keys = []

    # Encrypt each chunk
    for i in range(0, len(binary_message), chunk_size):
        binary_chunk = binary_message[i:i + chunk_size]
        encrypted_chunk, random_key = quantum_encrypt_chunk(binary_chunk)
        encrypted_chunks.append(encrypted_chunk)
        random_keys.append(random_key)

    # Combine the encrypted chunks
    encrypted_binary_message = ''.join(encrypted_chunks)
    encrypted_message = binary_to_string(encrypted_binary_message)

    return binary_message, encrypted_message, random_keys

# Android-specific interface functions
def android_encrypt(message: str) -> dict:
    """Wrapper function for Android to call encryption"""
    try:
        binary, encrypted, keys = quantum_encrypt(message)
        return {
            "success": True,
            "binary": binary,
            "encrypted": encrypted,
            "keys": keys
        }
    except Exception as e:
        return {"success": False, "error": str(e)}

def android_decrypt(encrypted_message: str, keys: list) -> dict:
    """Wrapper function for Android to call decryption"""
    try:
        decrypted = quantum_decrypt(encrypted_message, keys)
        return {
            "success": True,
            "decrypted": decrypted
        }
    except Exception as e:
        return {"success": False, "error": str(e)}   in this file which is quntum_encryption.py iam getting errors saying NO python interpreter configured for the module  ,  and oh the last file i should tell you about that as well so my MainActivity.kt which  is under   app>src>main>java>com.example.quantumencryptionapp>    also having some errors maybe becauce you told me to change the file soucrce earlier or may be not i dont know  so the errors are in it   Unresolved reference: material3    Unresolved reference: Text Unresolved reference: TextField Unresolved reference: it Unresolved reference: Text Unresolved reference: Button Unresolved reference: Text Unresolved reference: Text Unresolved reference: Button Unresolved reference: Text  Unresolved reference: Text Unresolved reference: CircularProgressIndicator   so now i have given you all the info now give me solutions and please provide full codes when you provide codes please give me codes that i can direct copy and past to to my system meaning give me codes that i can replace the current code present inmy system# Quanutm_encrypt
i have been trying to create an app for my project but stuck can anyone help
