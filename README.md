# fixWrap

Fix wraparound artifacts using c3d and the facemasking tool.

Wrapping in the phase direction (A-P) causes bad face masking.


## c3d

A customized c3d is needed (https://github.com/pyushkevich/c3d/pull/15). Eventually this will be merged into the c3d main build.


## Procedure

singularity exec `dcm_nii` and `analyze2dcm` in the facemasking container, or open a singularity shell. Mount a folder where you can see the original data.

```
# In container, run
For all structural series i
  dcm_nii ${i} ${i}_orig

# Outside container, run this
# Figure out how many slices need wrapping with ITK-SNAP
wrapSlices=20
# We remove one slice first with -pad and add it back after. This removes the blank slice at the front of the image
c3d ${i}_orig.nii -pad -1x0x0 0x0x0 0 -wrap ${wrapSlices}x0x0 -pad 1x0x0 0x0x0 0 -o ${i}_corrected.hdr

# Back in the container
For all structural series i
  # Note -y and -z flip here. VERIFY OUTPUT AGAINST ORIGINAL ANATOMICAL ORIENTATION
  analyze2dcm -o DICOM_${i}_corrected -y -z ${i} ${i}_corrected
