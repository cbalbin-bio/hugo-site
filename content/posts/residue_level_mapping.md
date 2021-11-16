---
author: "Christian Balbin"
title: "Residue Level Mapping without using SIFTS"
date: "2019-03-05"
description: "Guide to emoji usage in Hugo"
tags: ["Structural Biology"]
---

Often, I need to extract the protein sequence from a PDB structure. At first glance this may seem like a very straight forward task. But there are several pitfalls we need to be aware of.

### Extracting the residues from the atomic data yields unnatural sequences 

Your first thought, as was mine when I first started in this field, may be to simply extract the residues from the atomic data. The issue is this will yield unnatural sequences due to unmodeled (disordered) regions in the structure:

Take for example PDBID: 2GNK

  <script src="/ngl.js"></script>
  <script>
    document.addEventListener("DOMContentLoaded", function () 
        {



        if (body.classList.contains('colorscheme-dark'))
        {
            var backgroundColor= "#212121"
        } else {
            var backgroundColor= "#fafafa"
        }



        var stage = new NGL.Stage("viewport", { backgroundColor: backgroundColor });

        window.addEventListener("resize", function (event) 
            {
            stage.handleResize();
            }, false);


        darkModeToggle.addEventListener('click', () => {
        
            if (body.classList.contains('colorscheme-dark'))
            {
                stage.setParameters( { backgroundColor:"#212121"});
            } else {
                stage.setParameters( { backgroundColor:"#fafafa"});
            }

        });
        

        stage.loadFile("/2GNK.cif").then(function (o) 
            {
            o.addRepresentation("cartoon")
            o.addRepresentation("distance", {atomPair: [["37.CA","55.CA"]], color: "green",})
            o.autoView()
            });
    
        });



  </script>
  <div id="viewport" style="height:400px;"></div>

###### PDBID: [2GKN](https://www.rcsb.org/structure/2GNK) | Unmodeled region (residues 38-54) shown in green

The region in green is a disordered unmodeled region (NOTE: disordered and unmodeled do exactly mean the same thing but it is often the case that a disordered region is unsolved/unmodeled). Thus, because it is unsolved there are no atomic coordinates present for this region. If we were to extract the residues from the coordinate data, it would skip this region because it is not represented in the atomic data.

For example, notice that in the [PDB file](https://files.rcsb.org/view/2GNK.pdb) it jumps from residue 37 to 55.
```text
...
ATOM    274  N   GLY A  37     -15.075  39.805   6.661  1.00 44.73           N  
ATOM    275  CA  GLY A  37     -16.284  39.652   5.872  1.00 48.53           C  
ATOM    276  C   GLY A  37     -17.423  38.816   6.407  1.00 50.47           C  
ATOM    277  O   GLY A  37     -17.543  37.645   6.067  1.00 56.21           O  
ATOM    278  N   PHE A  55     -13.565  41.983  11.476  1.00 42.29           N  
ATOM    279  CA  PHE A  55     -12.505  40.970  11.325  1.00 43.12           C  
ATOM    280  C   PHE A  55     -12.714  39.735  12.211  1.00 43.54           C  
ATOM    281  O   PHE A  55     -12.850  39.879  13.442  1.00 47.11           O  
ATOM    282  CB  PHE A  55     -11.171  41.606  11.629  1.00 44.34           C  
...
```

Therefore, while the true expressed sequence of the structure would look like this (truncated to +/- 5 residues around disordered region for explanatory purposes)
```text
34   VKGFGRQKGNAELYRGAEYSVNFLPKV   59
```


While your extracted sequence would look like this
```text
34   VKGFG*****************FLPKV   59
```

If you are simply using the extracted sequence to map a structural representative to an alignment, this may not be such a problem as you don’t have structural data for that region in the first place, and you could let the alignment algorithm “figure it out” and add gaps to that region (albeit this is not a good practice).

But if you instead intend to use the extracted sequence as input to some predictor, the missing region becomes a significant issue as most predictors rely on the context around a residue. Feeding a predictor the sequence VKGFGFLPKV will generate inaccurate results as it no longer has the context of the residues of the unmolded disordered region that actually exist in the true sequence.

### SEQRES does not need to match the ATOM records 

Okay, so extracting the residues from the coordinate data has its problems. What about the SEQRES header or provided fasta file from the PDB website.


The SEQRES header is supposed to contain the “primary” sequence used in the experiment. The actual expressed protein is derived from the “primary” sequence. This means that the solved protein can be a fragment, extended or mutated version of the “primary” sequence.

Number 6 atomic in will not equal residue number 6 in SEQRES example 5TFR.

Often the terminal regions of a protein will be disordered (unmodeled).

 <!-- This is useful because you may build an alignment inlcluding the seqeuence of the PDB structure, and then you can use that structure as a structural representative for sites in the alignments. -->


Extracting the sequence that composes a PDB structure is not so simple though. The SEQRES header does not correspond 1:1 to the residues in the structure. You can extract the amino acids that build up the structure but now you may be left with a sequence that is missing regions because they were not solved (disordered) in the structure. If you are only extracted the sequence for use as a structural representative in an alignment, this may not be such a problem as usually the alignment algorithm will “figure it out” and add gaps in those missing regions. But if you intend to use the extracted sequence to benchmark predictors, the missing regions may be an issue, because the extracted sequence does not represent a true biological sequence due to the missing regions.