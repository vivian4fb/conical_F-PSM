# Curved waveguide with holographic couplers, rebuilding the Ansys KBA

Prompts for Sonnet 5.5, run from this folder. Paste one, let it finish, read what it actually did,
then paste the next. If a step didn't work, sort it out before moving on. The later ones lean on
the earlier ones.

The KBA: https://optics.ansys.com/hc/en-us/articles/43789665327123-Curved-Waveguide-Design-with-Holographic-Couplers

Two things I have to do by hand first, because the Ansys site refuses scripts:

1. Open the KBA in a browser, print it to PDF, save it as `source/KBA_curved_waveguide.pdf`.
2. Click "download example" on the same page and put the zip in `source/` as it comes. Don't unzip it.

OpticStudio 2026 R1 is on this PC. Lumerical isn't (checked 2 Nov 2026), so the RCWA part gets
packed up and run on the other machine, same as the F-PSM TM pack.


## 1. Read it, build nothing

```text
I'm working in C:\Users\vivia\Code2\1_Optics\Synopsys\VHG. I want to rebuild the Ansys KBA "Curved Waveguide Design with Holographic Couplers" myself and get the same answers they got.

The article is in source/KBA_curved_waveguide.pdf and their example zip is next to it. Read the whole PDF properly, then list what's in the zip. Don't extract anything outside source/.

Don't build anything yet. All I want from this step is one file, notes/kba_numbers.md, with every number the article gives. Wavelength(s), waveguide material and index, thickness, inner and outer radius, length, where each coupler sits, the construction points for each hologram, the toroidal radii, hologram thickness, n0, delta n, FOV, eyebox, detector size, ray counts. Page number next to each one. If a value only shows up in a screenshot, write "screenshot, p.X" so I know it was read off an image.

If something on my list isn't in the article, write "not given" and leave it. Don't fill gaps with typical values. I'd rather have a hole than a guess.

Then a few lines on the order they do things in. I think it goes: ideal toroidal holograms in OpticStudio first, then Kogelnik efficiency, then RCWA in Lumerical with layer repetition, then the results back into OpticStudio through LSWM. Tell me if I've got that wrong.
```


## 2. What's actually in their files

```text
Now their example files. Unpack each .zar into work/kba_original/ and leave source/ alone. If you can't unpack a zar from the API, stop and tell me, I'll do it by hand.

For every lens file, go through the non-sequential editor with ZOS-API, not by reading the file as text, and dump every object to a CSV in notes/: type, position, tilts, material, every parameter, and the diffraction settings where there are any. For the Lumerical files you can't open them here, so just tell me what each one is, which solver, and whether there's a script inside.

Then check everything in notes/kba_numbers.md against the files. Where the article and the file disagree, the file wins, but flag it. I only want the disagreements, as a table.

Use 1_Optics\.venv, don't make a new one. OpticStudio's path comes from 0_System/env.ps1, don't type it in.
```


## 3. Boring checks

```text
Before any physics, a few checks. Short answers.

1. What licence tier is OpticStudio on here? The user-defined diffraction DLLs need Premium or Enterprise, and I think LSWM does too. If we're on Professional I need to know now, because then the LSWM step goes in the remote pack as well.
2. Is lumerical-sub-wavelength-2026R1.dll still in Documents\Zemax\DLL\Diffractive?
3. Does the Toroidal hologram object the KBA uses exist in 2026 R1 under the same name, with the same parameters?
4. Confirm Lumerical really isn't installed on this PC.

Answers go in notes/environment.md, with today's date on each. Anything you couldn't check is UNTESTED. Don't assume it's fine.
```


## 4. Back of envelope before the big model

```text
Quick sanity pass in Python before we open OpticStudio properly. One small script in code/, nothing clever.

From the numbers in notes/kba_numbers.md:

- The range of propagation angles inside the guide across the full FOV after the in-coupler. Check every field is past the critical angle on both faces. If any field doesn't TIR, tell me which one.
- Bounce length (inner face to outer face and back) for the centre and edge fields, against the out-coupler pitch. That's usually where gaps in the eyebox come from.
- How much the guide itself focuses a collimated bundle in the curved direction, per bounce and over the whole path to the last out-coupler. Use Coddington, and be careful which plane is which. Both faces count, and they don't cancel because the radii differ by the thickness. Outer radius is 171.45 mm, the rest comes from the notes.

Print everything with units. Then two lines on whether the article's layout agrees with these numbers.
```


## 5. The problem: curved guide, no correction

```text
Right, OpticStudio. Build our own model in work/, through ZOS-API, from the numbers in the notes. Don't just resave theirs, theirs is what we check against.

First the case the article uses to show the problem. Curved guide, outer radius 171.45 mm, couplers with no extra power, construction points pushed far out so every hologram is effectively a straight grating. Collimated input.

They show that the rays inside the guide aren't collimated any more. I want to see the same thing:
- 3D layout, saved as png in figures/
- detector image at the eye
- the same two from their file, next to ours

If ours doesn't look like theirs, stop and tell me why before going on. Don't tweak things until it matches.

This is config 1. Never hand edit the lens file.
```


## 6. The fix: toroidal holograms

```text
Now the correction. Swap the in-coupler and the three out-couplers for the ideal Toroidal hologram objects, like the KBA does, and use the toroidal curvature to put power into the holograms in the curved (sagittal) direction to cancel what the guide is doing. Construction point far away means a collimated recording beam, close in means a spherical one. That's how the power gets in.

Use the KBA's construction points and radii exactly, first. Get their image. Then, separately, compare the power they put in against what you worked out in step 4. Close is fine. If it isn't close, give me the difference in dioptres.

Same outputs as step 5, plus an RMS spot size at the eye for the centre and four corner fields, ours against theirs. Save it as config 2 in the same file. Leave config 1 alone.
```


## 7. Kogelnik efficiency

```text
Switch on the Kogelnik efficiency for the volume holograms, using the Volume Hologram parameters from the KBA. Run it.

What I want to know:
- how much light comes out of each of the three out-couplers, across the field
- uniformity across the eyebox as max over min
- total throughput, source to eye, in percent

Kogelnik assumes the hologram index matches whatever's around it. Write one line on whether that holds here and roughly how far off it'll be. The RCWA step is there to answer that properly, so don't spend long on it.
```


## 8. Prove the RCWA route on the easy case first

```text
Before the real couplers go anywhere near Lumerical, I want the RCWA route checked on a case we already know the answer to.

The other Ansys article (VHG with RCWA and layer repetition) is saved here:
1_Optics/Research/final_results/evidence/ansys_articles/ansys_article_37118679173267_VHG_lumerical.txt
Its model is 1_Optics/Research/17-09 files/lumerical/ansys_example/vhg_R_6um_model.fsp and the OpticStudio test is VHG_test_R.zar in the folder under it.

Their case: n0 = 1.5, delta n = 0.04, 6 µm thick, recorded at 550 nm with beams at 0° and 28° inside the material (0° and 44.7° in air), reflection type. That gives px = 0.781 µm and pz = 0.1947 µm, sliced into 10 layers per period, and they get about 80 % into R+1 at Bragg for s-pol.

Run that case through our Python Kogelnik and check we land near 80 %. There's already one from the F-PSM work in Research/2026-10-16/remote_pack/code/kogelnik.py. Use that, don't write another. This number becomes the gate on the Lumerical PC in the next step.
```


## 9. The real couplers in RCWA (remote pack)

```text
Now the actual couplers from the KBA.

The holograms have optical power, so the grating isn't the same everywhere across a coupler. The KBA samples the hologram function at points across each coupler, runs RCWA at each sample with layer repetition, and exports the lot as LSWM with spatial variation. The out-couplers are transmission gratings. The in-coupler I'm not sure about, check the article.

Lumerical is on the other PC, so build me a pack in remote_pack/ that I can run there with Claude. Copy the layout of Research/2026-10-16/remote_pack: a CLAUDE.md that says start here, one execution brief, the scripts, the reference articles, and a RETURN.md that says exactly which files come back and where they go.

The brief needs:
- the sample grid for each coupler, and why that spacing is fine (show how much the local period changes between neighbouring samples)
- angle and wavelength ranges that cover the whole FOV inside the guide, from step 4
- the layer repetition setup
- a gate: the 550 nm case from step 8 has to come out near 80 % R+1 on that PC before any coupler runs
- export as .json, the legacy LSWM format, unless you've confirmed the 2026R1 DLL here reads .lswm

Don't run anything, just build it. Zip it when it's done and tell me the size.
```


## 10. LSWM back into OpticStudio

```text
The LSWM files are back. They're wherever RETURN.md said they'd go.

Check them before using them. Each one loads, the sample grid is what we asked for, and at a couple of sample points the efficiency at Bragg is near the Kogelnik number from step 7. Tell me where they differ and by how much, in percentage points.

Copy them into Documents\Zemax\DLL\Diffractive. Then config 3: same guide, with the toroidal holograms swapped for whatever object the KBA uses to read the LSWM data. Leave configs 1 and 2 as they are.

Run it and give me one table: RMS spot at centre and corners, output per out-coupler, eyebox max/min, throughput. Configs 2 and 3 side by side, and against the KBA's numbers. Every number with units and its difference from the KBA.
```


## 11. Efficiency against bandwidth

```text
The article says you have to trade peak efficiency against how flat the response is over the angular bandwidth. I want to see that trade for our out-couplers, not just take their word for it.

For the three out-couplers, sweep delta n and thickness around the KBA's values (Kogelnik is fine here, it's only for the shape of the trade) and plot peak efficiency against angular bandwidth over the FOV. Mark where the KBA's choice sits. For equal output from three couplers in a row the ideal efficiencies are 1/3, 1/2 and 1, so show how close each one gets.

One plot, one short paragraph. Don't redesign anything.
```


## 12. Write it up

```text
Last one. REPORT.md in this folder.

Open with one bold line: did we reproduce the KBA or not. Then a table with every result they show, our number, their number, the difference, and a status: VERIFIED with today's date if we measured it, UNTESTED if we didn't get to it. Anything we couldn't reproduce gets its own short section with the reason. Don't soften it.

Then update context.md in this folder with what's actually there now, the acceptance gate, and what's left. Real state only.

British spelling throughout.
```


## If it goes sideways

Paste one of these as it is.

```text
Where did that number come from? Show me the page in the PDF or the line in the file.
```

```text
That's a typical value, not theirs. Take it out and mark it not given.
```

```text
You've edited the lens file by hand. Put it back and do it through the API.
```

```text
Stop. Don't change things to make it match. Tell me why it doesn't match first.
```

```text
Too long. Table and verdict, nothing else.
```


## Where the numbers in here come from

I couldn't open the KBA page from here (403 to anything that isn't a browser), so prompt 1 makes
Sonnet read it from the PDF. Everything above was pinned down like this on 2 Nov 2026:

| What | Source | Status |
|---|---|---|
| 171.45 mm outer radius | Draper and Blanche abstract (Europe PMC), and the KBA summary in search results. Same number in both. | VERIFIED 2026-11-02 |
| 1 in-coupler + 3 out-coupling VHGs, toroidal holograms first, construction points far/near, Kogelnik through Volume Hologram parameters, RCWA with layer repetition, LSWM with spatial variation, transmission out-couplers, efficiency vs uniformity trade | KBA, but only through search-engine excerpts. Not read in full. | ASSUMED until prompt 1 confirms it |
| n0 1.5, Δn 0.04, 6 µm, 550 nm, 0°/28° (44.7° in air), px 0.781 µm, pz 0.1947 µm, 10 slices/period, ~80 % R+1 | Companion KBA 37118679173267, local copy read in full | VERIFIED 2026-11-02 |
| 1/3, 1/2, 1 for equal output from three couplers | Arithmetic, lossless. Coupler k takes 1/(N−k+1) of what's left. | VERIFIED 2026-11-02 |
| Lumerical not on this PC, OpticStudio 2026 R1 is, LSWM 2026R1 DLL in Diffractive | Checked the disk | VERIFIED 2026-11-02 |
| OpticStudio licence tier | Not checked. Prompt 3 does it. | UNTESTED |

The paper behind the KBA is Draper and Blanche, "Holographic curved waveguide combiner for HUD/AR
with 1-D pupil expansion", Opt. Express 30(2), 2503–2516 (2022), doi 10.1364/OE.445091. Paywalled.
Its abstract gives 13° × 16° FOV at 1 cm, 5× horizontal expansion and a 6.2 mm × 42.7 mm eyebox.
The KBA uses three out-couplers, so its numbers may not be the paper's. Don't mix the two.
