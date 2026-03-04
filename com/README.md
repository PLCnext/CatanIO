
# 📘 CatanIO – API Documentation for RS-485 (ComMessage)

## Overview

This API defines the RS‑485 communication for Emalytics CatanIO modules.
It supports configuration, transmitting (TX), receiving (RX), and general communication parameters.
The RS‑485 stack is based on serial bus characteristics such as baud rate, parity, and data bits, and is configured through CoAP endpoints.

## Dependencies
- `shared.proto` → Enums, settings, helper structures
- `uoi.proto` → UOI-specific messages and enums

---

# Endpoints

| Method | Example URL | Description |
|--------|-------------|-------------|
| `GET`  | `/com/<channel>/cfg` | Read configuration (`RespConfig`) |
| `PATCH`| `/com/<channel>/cfg` | Write configuration (`ReqWriteConfig`) |
| `PUT`  | `/com/<channel>/tx`  | Send data (`ReqTx`) |
| `GET`  | `/com/<channel>/rx`  | Receive data (`ReqRx`) |
| `PATCH`| `/com/<channel>/rx`  | Acknowledge reception (`RespRx`) |

All messages are exchanged using the shared `ComMessage` structure.

---

# Messages and Functions

## ComMessage
The main structure consists of the following components:

- Configuration requests and responses
- TX requests and responses
- RX requests and responses
- Complete serial parameters (baud rate, parity, etc.)

---

# Configuration

## ReqWriteConfig – Configuration write
| Field      | Type            | Description |
|------------|-----------------|-------------|
| enabled    | BooleanSetting  | Interface enabled/disabled |
| baud       | Baud            | Predefined baud rate |
| user_baud  | uint32          | User-defined baud rate |
| data_bits  | DataBits        | Number of data bits |
| parity     | Parity          | Parity mode |
| stop_bits  | StopBits        | Stop bits |
| name       | String64Setting | Channel name |

---

## RespConfig – Read configuration (GET) or after WriteConfig

| Field | Type | Description |
|------|------|-------------|
| enabled | bool | Enabled / disabled |
| use_default_val_on_start | bool | Use default value on system startup |
| use_default_val_on_failure | bool | Use default value on failure |
| default_val | bool | Default value |
| override_enabled | bool | Override allowed |
| override_cancelable | bool | Override cancelable |
| name | string | COM channel name |

---

# Sending (TX)

## ReqTx

| Field | Type | Description |
|------|------|-------------|
| data | bytes (max 400) | Raw data transmitted via RS‑485 |

---

## RespTx

| Field | Type   | Description |
|------|--------|-------------|
| err  | uint32 | Error code (0 = OK) |

---

# Receiving (RX)

## ReqRx – Receive request

| Field | Type | Description |
|------|------|-------------|
| min_len | uint32 | Minimum number of bytes required to trigger reception |
| timeout_ms | uint32 | Timeout (ms) |
| aggregation_timeout_bit_times | uint32 | End-of-packet detection |
| max_len | uint32 | Maximum length of the RX response |

---

## RespRx – Received data

| Field | Type | Description |
|------|------|-------------|
| err  | uint32 | Error code |
| data | bytes (max 400) | Received data |

---

# ComMessage.MsgType

| Value | Name | Description | Payload |
|------|------|-------------|---------|
| 0 | TYPE_REQ_READ_CONFIG | Read configuration | RespConfig |
| 1 | TYPE_RESP_READ_CONFIG | Response to read request | RespConfig |
| 2 | TYPE_REQ_WRITE_CONFIG | Write configuration | ReqWriteConfig |
| 3 | TYPE_RESP_WRITE_CONFIG | Response to write request | RespConfig |
| 4 | TYPE_REQ_RX | Request reception | ReqRx |
| 5 | TYPE_RESP_RX | Reception response | RespRx |
| 6 | TYPE_REQ_TX | Send data | ReqTx |
| 7 | TYPE_RESP_TX | Response to TX | RespTx |

---

# Parameter enums

## Baud
| Value | Name |
|------|------|
| 0 | BAUD_DONT_CHANGE |
| 1 | BAUD_DEFAULT |
| 2 | BAUD_USER |
| 3 | BAUD_50 |
| 4 | BAUD_75 |
| 5 | BAUD_110 |
| 6 | BAUD_134 |
| 7 | BAUD_150 |
| 8 | BAUD_200 |
| 9 | BAUD_300 |
| 10 | BAUD_600 |
| 11 | BAUD_1200 |
| 12 | BAUD_1800 |
| 13 | BAUD_2400 |
| 14 | BAUD_4800 |
| 15 | BAUD_9699 |
| 16 | BAUD_19200 |
| 17 | BAUD_38400 |
| 18 | BAUD_57600 |
| 19 | BAUD_76800 |
| 20 | BAUD_115200 |

---

## DataBits
| Value | Name |
|------|------|
| 0 | DATA_BITS_DONT_CHANGE |
| 1 | DATA_BITS_DEFAULT |
| 2 | DATA_BITS_5 |
| 3 | DATA_BITS_6 |
| 4 | DATA_BITS_7 |
| 5 | DATA_BITS_8 |

---

## Parity
| Value | Name |
|------|------|
| 0 | PARITY_DONT_CHANGE |
| 1 | PARITY_DEFAULT |
| 2 | PARITY_NO |
| 3 | PARITY_EVEN |
| 4 | PARITY_ODD |
| 5 | PARITY_MARK |
| 6 | PARITY_SPACE |

---

## StopBits
| Value | Name |
|------|------|
| 0 | STOP_BITS_DONT_CHANGE |
| 1 | STOP_BITS_DEFAULT |
| 2 | STOP_BITS_1 |
| 3 | STOP_BITS_2 |

---


