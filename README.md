# sip_call — C SIP calling application (liblinphone)

A minimal but complete SIP phone that can both **place and receive calls**,
built in C on top of Belledonne's `liblinphone`.

---

## Prerequisites (WSL / Ubuntu 22.04+)

### 1. Install build tools

```bash
sudo apt update
sudo apt install -y build-essential cmake pkg-config git
```

### 2. Install liblinphone

The easiest route on Ubuntu/Debian is the official Belledonne PPA:

```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:belledonne-communications/nightly
sudo apt update
sudo apt install -y liblinphone-dev
```

If the PPA is unavailable, install the latest stable release manually:

```bash
# Download the .deb from https://www.linphone.org/technical-corner/linphone
# or build from source: https://github.com/BelledonneCommunications/linphone-sdk
```

### 3. Audio in WSL

WSL 2 (Windows 11, build 22000+) supports PulseAudio via WSLg automatically.
For older setups install PulseAudio manually:

```bash
sudo apt install -y pulseaudio
pulseaudio --start
```

Check devices are visible to linphone:
```bash
pactl list short sources
pactl list short sinks
```

---

## Build

```bash
cd sip_call
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
```

The binary lands at `build/sip_call`.

---

## Usage

```
./sip_call <sip_user> <sip_password> <sip_domain>
```

**Example** — connect to a free SIP provider (e.g. sip.linphone.org):

```bash
./sip_call alice mysecret sip.linphone.org
```

### Commands at the prompt

| Command                       | Action                          |
|-------------------------------|---------------------------------|
| `call sip:bob@example.com`    | Place an outgoing call          |
| `answer`                      | Accept an incoming call         |
| `hangup`                      | End / reject the current call   |
| `status`                      | Print registration & call state |
| `quit`                        | Unregister and exit             |

---

## Testing without a real SIP server

Use a local **Asterisk** or **Kamailio** instance, or try these free public
registrars:

| Provider               | Domain                  | Register URL           |
|------------------------|-------------------------|------------------------|
| Linphone.org           | `sip.linphone.org`      | https://www.linphone.org |
| OnSIP (free trial)     | `app.onsip.com`         | https://www.onsip.com  |
| IPtel.org              | `iptel.org`             | https://www.iptel.org  |

---

## Project structure

```
sip_call/
├── CMakeLists.txt
├── README.md
└── src/
    └── main.c          ← all application logic
```

### Key code sections in main.c

| Function / section        | Purpose                                           |
|---------------------------|---------------------------------------------------|
| `setup_account()`         | Creates proxy config + auth info, starts REGISTER |
| `cb_registration_state()` | Prints registration events to console             |
| `cb_call_state()`         | Handles all call lifecycle events                 |
| `cmd_call()`              | Sends SIP INVITE via `linphone_core_invite()`     |
| `cmd_answer()`            | Accepts incoming call with `linphone_call_accept_with_params()` |
| `cmd_hangup()`            | Sends BYE via `linphone_call_terminate()`         |
| Main loop                 | `linphone_core_iterate()` every 20 ms + `select()` for stdin |

---

## Troubleshooting

**"No audio"** — check PulseAudio is running (`pulseaudio --check -v`).

**"Registration failed"** — verify credentials and that the SIP port (5060/UDP
or 5061/TLS) is not blocked by your firewall.

**Compile error: `linphonecore.h` not found** — run `pkg-config --cflags liblinphone`
to confirm the library is installed and visible.

**WSL firewall** — Windows Defender may block UDP 5060. Add an inbound rule:
```
netsh advfirewall firewall add rule name="SIP UDP" protocol=UDP dir=in localport=5060 action=allow
```
