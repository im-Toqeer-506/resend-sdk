# Resend SDK generation report

Date: 16 October 2026  
Author: M Toqeer Zia

## Outcome

I generated an MIT-licensed TypeScript SDK for the Resend public API with the
Voxgig SDK generator. The repository contains the generator project, the
source OpenAPI document, generated TypeScript source and declarations,
offline tests, API reference documentation, CI workflows, and publishing
workflow scaffolding.

The generated SDK models 139 operations across 79 semantic entities. Its test
mode replaces HTTP with an in-memory transport, so consumers can test their
integration without credentials or network access.

## Why Resend

- Resend has a free plan and an API-key-based API suitable for an SDK trial.
- Resend publishes its OpenAPI 3.1 specification in the
  [`resend/resend-openapi`](https://github.com/resend/resend-openapi)
  repository.
- A query of the live Voxgig SDK catalog for `resend` returned no results on
  16 October 2026, so the API met the assignment's catalog requirement.

The checked-in specification reports API version `1.5.1` and was downloaded
from upstream commit `a210174fb0b07f077ba7c3d43fefe56b06500b14`.

## Reproduce

The project uses Node.js 24, recorded in `.nvmrc`.

```bash
cd .sdk
npm install
npm run generate

cd ../ts
npm install
npm test
```

The scaffold was created with:

```bash
npx --yes @voxgig/create-sdkgen@0.30.2 resend \
  --def /path/to/resend-openapi.yaml \
  --target ts
```

Installed generation components included `@voxgig/sdkgen@4.32.1`,
`@voxgig/apidef@8.20.0`, and `@voxgig/model@12.0.1`.

## Verification

- SDK generation: passed.
- TypeScript compilation: passed.
- Generated offline suite: 139 tests passed, 0 failed.
- Dependency audit: 0 known vulnerabilities in both the generator and SDK
  dependency trees at installation time.
- Live authenticated behavior: not executed because no Resend API key was
  available in the environment. No email was sent and no remote account data
  was changed.

## Developer experience

The strongest part of the workflow was the amount of usable project structure
produced from one specification. The generator created typed entities,
documentation, CI configuration, a publishing workflow, and a substantial
offline suite. Keeping `.sdk/` in the repository also makes the result
reproducible instead of treating generated output as an opaque one-off.

The entity-oriented API is interesting and keeps raw routes out of application
code. The generated `test` feature is especially useful because it makes SDK
consumer tests deterministic and credential-free.

There were four points of friction:

1. The initializer treated an HTTPS value passed to `--def` as a local path,
   so the OpenAPI document had to be downloaded first.
2. Current transitive generation dependencies declare Node.js 24 as their
   minimum, although the introductory documentation said Node.js 22 or later.
   Generation happened to succeed on Node.js 22.23.3, but the repository now
   declares Node.js 24 to avoid relying on unsupported behavior.
3. The semantic heuristic produced some awkward entity names based on response
   schemas, such as `AddContactToSegmentResponseSuccess` and
   `UpdateEmailOption`. A production pass should add model guidance so common
   Resend concepts have a smaller, more idiomatic surface.
4. Generation emitted non-fatal `require-missing` warnings for
   `ReadmeFeatures_ts` and `AgentGuide_ts`. Output and tests were still
   complete, but warnings this specific should include a short explanation or
   remediation hint.

Overall, the generator delivered a working, documented SDK quickly. The next
production-hardening steps would be refining the inferred entity model,
running the live contract suite with a dedicated least-privilege Resend test
key, and publishing the package after repository ownership is confirmed.
