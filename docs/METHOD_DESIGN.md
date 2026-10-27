# Method design: self-calibrating inverse rendering for VBTS (draft v1, 2026-10-27)

Status: design for review. Nothing implemented. Replaces the prototype `scripts/tacrep_factor.py`
(stopped before its first evaluation; it was a minimal swap autoencoder without physical structure).

## 1. Problem

A VBTS image of a touch is a rendering: an opaque, coated membrane deforms against a surface, and
three coloured LEDs shade the deformed membrane for a camera. Everything the image says about the
touch is in the membrane's geometry; everything else is the sensor. We measured that the "everything
else" is a property of the hardware (unit + gel), stable over months, abrupt at hardware changes, and
that classifiers trained with one object per session learn it as a class feature. The subtraction
fix assumes the sensor adds to the image. It does not model the sensor's photometric response (how
a surface slope becomes a colour), which differs between units, nor the gel's mechanics.

Goal: a touch representation that is invariant to the hardware by construction, inferred from a
single contact image at test time, with a hardware model that the robot calibrates from its own
unlabelled contacts (no calibration ball, no blank frame required, no labels), and that tells us
when a hardware change is mechanical rather than optical.

## 2. Image-formation model (the physics we commit to)

For hardware state h (unit, gel) and a contact crop at frame position p:

    x(p) = B_h(p) + A_h(p) ⊙ R_h(p, ∇z(p)) + ε

- z(p): height field of the membrane (the touch), ∇z its gradient. Normals are a deterministic
  function of ∇z, so integrability holds by construction.
- R_h(p, g): the photometric response, mapping a surface gradient g at position p to an RGB change.
  This is exactly what the manufacturer's per-pixel calibration learns once, with a ball press, for
  one reference unit (gsrobotics `RGB2NormNet`: (r, g, b, x, y) -> (gx, gy); shipped as
  `third_party/gsrobotics/models/nnmini.pt`). We learn it per hardware, without a ball.
- B_h(p): the bare-gel image (illumination, optics, the fixed fine pattern we measured). Observed
  directly where bare-gel frames exist (S2, S3); latent elsewhere.
- A_h(p): a smooth gain field (LED falloff, coating).
- The gel's mechanics enter z: the same surface pressed with the same force gives a different z on
  a gel of different stiffness. We model this as a per-gel scale/compliance s_h on z (z = s_h · u,
  u the hardware-free imprint), testable with the Nano17 force.

## 3. Model

1. **Hardware model** (one per hardware state, or amortised from a stream):
   - B_h: explicit full-resolution parameter image (1232x1640x3), initialised from the mean of the
     hardware's contacts (our state field) or its bare-gel frames; regularised to the bare-gel frames
     where available.
   - A_h, R_h: a small conditional MLP / neural field R(p, g; e_h) with a hardware embedding e_h
     (FiLM), smooth in p (Fourier features of low bandwidth), monotone-ish in g; initialised from the
     manufacturer MLP. A_h low-frequency (B-spline grid).
   - s_h: scalar compliance per gel.
2. **Touch encoder** E: x -> (u, ∇u) — a height field on the crop, single image, hardware-agnostic
   input but trained so that its output is hardware-invariant (Section 4). Native resolution; U-Net
   with the DetailPatchEncoder stem (no downsampling of the input).
3. **Renderer** x̂ = B_h + A_h ⊙ R_h(p, s_h ∇u): differentiable, exact given the parameters.
4. **Touch representation** for downstream tasks: a texture encoder T on the imprint u (and its
   gradient field), plus marker displacement for force.
5. **Hardware encoder** H (amortised calibration): a set encoder over N unlabelled contacts (and bare
   frames if any) of the current hardware -> e_h, B̂_h. Permutation-invariant attention pooling over
   per-image residual features; the objects average out because the set spans objects. Used for
   new hardware at test time and to detect hardware changes (distance in e_h).

## 4. Training (why the factorisation is identifiable, and how it is enforced)

Data: sessions grouped by hardware state (S1, S2, S3, known from the images). Losses, all label-free
except the optional downstream head:

- L_rec: x vs x̂ (L1 + high-pass L1 + SSIM on markers-inpainted crops).
- L_int: integrability is structural (z is the output); add a smoothness prior on u at scales below
  the measured sensor noise floor (~90 µm), nothing above it.
- L_shared (the key identifiability lever): the touch encoder is shared across hardware, and the
  distribution of imprints u is hardware-independent for the same materials. Where the same material
  is recorded on two hardware states (A/B on S1 vs D1 on S2 — available in folds A and B), align
  imprint statistics per material (MMD on u-features); where it is not (fold D), the shared encoder
  plus the physical renderer is the only constraint.
- L_swap: re-rendering a crop under another hardware's model, x' = B_h' + A_h' ⊙ R_h'(s_h' ∇u),
  must re-encode to the same u (cycle consistency in the touch latent), and must be judged
  consistent with h' by a patch discriminator conditioned on e_h' (realism of the counterfactual).
- L_bare: B_h close to bare-gel frames where they exist.
- Optional L_cls: material head on T(u) with counterfactual re-renders as extra training samples.

Identifiability argument (to be written formally): (i) integrability removes the non-integrable
part of the ambiguity; (ii) the remaining generalised bas-relief family (Belhumeur et al.) is shared
across hardware only if R_h are related by the same GBR transform, which a shared encoder plus
per-hardware responses fitted to the same material statistics excludes; (iii) the content/style
split follows the multi-view identifiability results (von Kügelgen et al. 2021; Locatello et al.
2020; Daunhawer et al. 2023) with hardware state as the group variable and paired views given by
same-material sessions across hardware and by the re-renders. To be checked by the review.

## 5. Inference

- Known hardware (seen in training): u = E(x), single image.
- New hardware, deployable: freeze E; fit (B_h, A_h, R_h via e_h, s_h) to the first N unlabelled
  contacts by analysis-by-synthesis (minimise L_rec), or amortised with H. No ball, no labels,
  no blank needed (uses them if present). Report accuracy vs N.
- Hardware-change detection: e_h of the current window vs the last; mechanical change flagged when
  the fitted s_h or marker stiffness changes.

## 6. Evaluation (rigour expected at CVPR)

- Splits: leave-one-campaign-out (A, B, C, D) and leave-one-hardware-out (S1 -> S2, S2 -> S3, ...);
  test sessions never in training; three seeds; paired bootstrap over materials.
- Geometry check: our recovered u vs the manufacturer-calibrated depth (gsrobotics) on the reference
  unit; consistency of u for the same material across hardware (the method's direct claim).
- Downstream: material (20-way), normal force (Nano17), contact; for each, cross-hardware.
- Baselines: raw CNN; photometric jitter; blank subtraction; contact-mean subtraction; MixStyle;
  RandConv; FDA (Fourier amplitude swap); DANN; GroupDRO; Tent/AdaBN; frozen Sparsh (+ background
  subtraction in its own protocol); calibration-free normals (SDM-UniPS / unipsgel2026); the
  manufacturer's normal model; SITR-style calibration if its data is public.
- Second sensor: Real Tactile MNIST (two gels, digits) and any public multi-unit set the audit finds.
- Ablations: no physical renderer (free decoder); no integrability; no shared-statistics loss; no
  swap; no discriminator; fixed vs fitted hardware model; N contacts for calibration; per-term R_h
  capacity.

## 7. Risks

- GBR/scale ambiguity may leave u hardware-dependent up to a transform; mitigated by the shared
  encoder and paired materials, measured by the geometry check.
- Markers: printed dots occlude; inpaint (as now) and mask in the loss.
- Fold D has only four sessions on S2 in training; the hardware model for S2 is fitted from them.
- Compute/time: native 192-px crops, 12 GB GPU. Deadline: abstract Nov 10, paper Nov 16.
