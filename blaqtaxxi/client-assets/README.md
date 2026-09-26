# client-assets/

**Keep the client's originals here locally. This folder is gitignored, and a pre-commit guard blocks it (`.githooks/pre-commit`). The scaffold does not ship the images. Never commit them, on any branch, in any repo, public or private (D-125).**

Put these files in this folder on your own machine:

| File | Where it goes |
|---|---|
| `wordmark__blaq.png` | Copied to `public/brand/` at build (Phase 0). 195×75, opaque, black; use on white only until a transparent/SVG and a reversed version are supplied (U-B4) |
| `favicon.jpg` | Favicon sizes generated into `public/brand/` (Phase 0). 512×512 JPEG, soft (U-B8) |
| `nissan-sentra.jpg` | Uploaded to the Sentra car record **through the admin** |
| `chevrolet-suburban-3500hdheavy-duty.jpg` | Uploaded to the Suburban car record through the admin. **Not a real car:** an example that shows the client he can manage a fleet and set a price per vehicle (U-V5) |
| `profile.jpg` | Uploaded as the driver photo through the admin; crop to the face (U-B7). Contains a real person: never commit, never show on camera |

The car images look like manufacturer stock images (unverified). Replace with the driver's real cars for a real launch (U-B5).

Uploads go to object storage (assumed Vercel Blob, D-113), not into git.
