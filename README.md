# rupis-static

Public static resources for the [Rupis](https://rupis.app) app. Apps fetch them
raw from GitHub:
`https://raw.githubusercontent.com/Karthikeyan-Karuppaiah/rupis-static/<branch>/…`

Nothing here is user data. The app only **downloads** these files, and only
when the user asks for it:

- **Remote catalog** (`config/<channel>/`): PSP, initiator, ads, MCC, purpose
  and payment lists plus icons. Fetched by Settings → Advanced → "Sync
  catalog".
- **On-device SMS model** (`ml/sms_ner/`): the INT8 ONNX token classifier that
  PRO users' SERVER-mode builds download once. After that, parsing runs fully
  on the device.

## Branches

| Branch | Read by | Holds |
|---|---|---|
| `main` | no app ("normal updates") | every channel |
| `store` | Play/App Store `rupis`, `rupisclone` | `store` only |
| `dev` | debug builds | `dev` only |
| `prof` | profile builds | `prof` only |
| `test` | `rupistest` | `test` only |
| `prod` | other release builds | `prod` only |

A release branch is `main` cut down to one channel. Changing `main` never
changes what any app sees until that channel's branch is released.

## Layout (per channel)

```
config/<ch>/manifest.json        signed catalog manifest (sha256 per file)
config/<ch>/manifest.json.sig    RSASSA-PKCS1-v1_5 / SHA-256, base64
config/<ch>/*.json, *_icons/     catalog files the manifest names
ml/sms_ner/<ch>/manifest.json    signed model manifest {key, id, bundle{path,sha256,bytes}}
ml/sms_ner/<ch>/manifest.json.sig
ml/sms_ner/<key>/rupis-ml-model  model bundle (ZIP_STORED: manifest.json, model.onnx, vocab.txt)
ml/sms_ner/<key>/LICENSE         the model's license
```

Each channel has its own signing key. The app checks the manifest against the
public key compiled into it, and every file against the sha256 in the signed
manifest. The host is not trusted, so a mirror of a branch works the same way.

## Third-party model license

`ml/sms_ner/<key>/rupis-ml-model` is an INT8-quantized ONNX export of
[FinancialSMS-NER](https://github.com/rkuma18/FinancialSMS-NER) by Roushan
Kumar. It is MIT-licensed, as the upstream README states. The full notice is in
`ml/sms_ner/<key>/LICENSE`, next to each bundle.

## Publishing (from the main rupis repo)

This repo is the `deploy/static` git submodule of the private app repo. It is
generated there, so do not edit it by hand:

```sh
task static:publish                       # regenerate main (all channels)
git -C deploy/static add -A
git -C deploy/static commit -m "publish: …"
git -C deploy/static push origin main     # still not live
FLAVOR=dev task static:release PUSH=1     # go live for one channel (store last)
git add deploy/static && git commit -m "chore(static): bump static host"
```

A model bundle is about 65 MB. That is under GitHub's 100 MB per-file limit, so
it needs no LFS (raw.githubusercontent.com would not serve LFS objects anyway).
Every branch reuses the same bundle blob, and each new model version adds one
bundle to the history.
