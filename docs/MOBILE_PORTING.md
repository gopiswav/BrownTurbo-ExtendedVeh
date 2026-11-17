# ExtendedVeh Android Mobile Porting Guide

Panduan teknis porting dan implementasi client-side **ExtendedVeh (Custom Handling, Sound, & Turbo System)** pada **Android SA-MP Mobile (`libGTASA.so` & `libsamp.so`)**.

---

## 1. Arsitektur Komponen

| Komponen | PC Windows | Android Mobile |
| :--- | :--- | :--- |
| **Arsitektur Binary** | `gta_sa.exe` (x86 32-bit PE) | `libGTASA.so` (ARMv7 / ARM64 ELF) |
| **Handling Table Pointer** | `mod_HandlingManager` (0xC2B9C8) | Symbol `mod_HandlingManager` via `dlsym(libGTASA, "mod_HandlingManager")` |
| **Network Packet** | RakHook packet 220 (`PKT_EXTENDEDVEH`) | Android RakClient hook packet 220 |
| **Vehicle Sound / Audio** | BASS / SA Engine Audio | BASS Audio / OpenSL ES |

---

## 2. Injeksi Dynamic Handling pada `libGTASA.so`

Pada Android, struktur `tHandlingData` memiliki padding yang disesuaikan dengan arsitektur compiler ARM:

```cpp
#include <dlfcn.h>
#include <cstdint>

// Dapatkan instance HandlingManager dari libGTASA.so
void* GetHandlingManager() {
    void* lib = dlopen("libGTASA.so", RTLD_NOLOAD);
    if (!lib) return nullptr;

    void* pManager = dlsym(lib, "mod_HandlingManager");
    return pManager;
}

// Update nilai handling mobil per vehicle ID
void ApplyCustomHandling(int vehicleModelId, float mass, float turnMass, float engineAcceleration, float maxVelocity) {
    void* pMgr = GetHandlingManager();
    if (!pMgr) return;

    // tHandlingData* pEntry = ((tHandlingData*(*)(void*, int))pGetHandlingData)(pMgr, vehicleModelId);
    // Modifikasi field handling sesuai konfigurasi server
}
```

---

## 3. Registrasi Packet Handler di Android RakClient

Saat menerima packet ID 220 dari server open.mp:
1. Dekode data serialisasi JSON/biner yang memuat properti `handling`, `turboLevel`, dan `soundOverride`.
2. Terapkan konfigurasi ke `CVehicle` target di memori `libGTASA.so`.
