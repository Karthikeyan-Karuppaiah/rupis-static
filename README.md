# rupis-static

Public static resources for the [Rupis](https://rupis.app) app, served by
GitHub Pages at `https://karthikeyan-karuppaiah.github.io/rupis-static/`.

Nothing here is user data. The app only **downloads** these files, and only
when the user asks for it:

- **Remote catalog** (`config/<env>/`): PSP, initiator, ads, MCC, purpose and
  payment lists plus icons. Fetched by Settings → Advanced → "Sync catalog".
- **On-device SMS model** (`ml/sms_ner/`): the INT8 ONNX token classifier that
  PRO users' SERVER-mode builds download once. After that, parsing runs fully on
  the device.

## Layout

```
config/<env>/manifest.json        signed catalog manifest (sha256 per file)
config/<env>/manifest.json.sig    RSASSA-PKCS1-v1_5 / SHA-256, base64
config/<env>/*.json, *_icons/     catalog files the manifest names
ml/sms_ner/<env>/manifest.json    signed model manifest {key, id, bundle{path,sha256,bytes}}
ml/sms_ner/<env>/manifest.json.sig
ml/sms_ner/<key>/rupis-ml-model   model bundle (ZIP_STORED: manifest.json, model.onnx, vocab.txt),
                                  content-keyed and shared by every env
```

`<env>` is one of `dev`, `prof`, `prod`, `test` or `store` (the Play Store
build). Each env has its own signing key. The app checks the manifest against
the public key compiled into it, and every file against the sha256 in the
signed manifest. The host is not trusted, so a mirror of this tree works the
same way.

## Third-party model license

`ml/sms_ner/<key>/rupis-ml-model` is an INT8-quantized ONNX export of
[FinancialSMS-NER](https://github.com/rkuma18/FinancialSMS-NER) by Roushan
Kumar. It is MIT-licensed, as the upstream README states. The full notice is in
`ml/sms_ner/<key>/LICENSE`, next to each bundle.

## Publishing (from the main rupis repo)

This repo is the `deploy/static` git submodule of the private app repo. It is
generated there, so do not edit it by hand:

```sh
task static:publish                      # every env; FLAVOR=dev,prod to narrow
git -C deploy/static add -A
git -C deploy/static commit -m "publish: catalog vN + model <key>"
git -C deploy/static push origin main
git add deploy/static && git commit -m "chore(static): bump static host"
```

A model bundle is about 65 MB. That is under GitHub's 100 MB per-file limit, so
it needs no LFS (and Pages would not serve LFS objects anyway). Each new model
version adds one bundle to the history, so only publish when the model changes.
