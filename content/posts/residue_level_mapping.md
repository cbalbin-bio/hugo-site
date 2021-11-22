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

But there is no gurantee that the Nth residue in the SEQRES header corresponds to the Nth residue in the cordinate data.
For example, the 6th residue in 5TFR's SEQRES HEADER is a glycine but the 6 residue in the atom data is a glutamate.



### _pdbx_poly_seq_scheme in mmCIF files contains a residue level mapping between the primary sequence and the cordinate data

```text
...
loop_
_pdbx_poly_seq_scheme.asym_id 
_pdbx_poly_seq_scheme.entity_id 
_pdbx_poly_seq_scheme.seq_id 
_pdbx_poly_seq_scheme.mon_id 
_pdbx_poly_seq_scheme.ndb_seq_num 
_pdbx_poly_seq_scheme.pdb_seq_num 
_pdbx_poly_seq_scheme.auth_seq_num 
_pdbx_poly_seq_scheme.pdb_mon_id 
_pdbx_poly_seq_scheme.auth_mon_id 
_pdbx_poly_seq_scheme.pdb_strand_id 
_pdbx_poly_seq_scheme.pdb_ins_code 
_pdbx_poly_seq_scheme.hetero 
A 1 1   GLY 1   -3  ?   ?   ?   A . n 
A 1 2   SER 2   -2  ?   ?   ?   A . n 
A 1 3   HIS 3   -1  ?   ?   ?   A . n 
A 1 4   MET 4   0   ?   ?   ?   A . n 
A 1 5   GLY 5   1   ?   ?   ?   A . n 
A 1 6   GLY 6   2   ?   ?   ?   A . n 
A 1 7   GLY 7   3   ?   ?   ?   A . n 
A 1 8   THR 8   4   ?   ?   ?   A . n 
A 1 9   GLY 9   5   ?   ?   ?   A . n 
A 1 10  GLU 10  6   6   GLU GLU A . n 
A 1 11  THR 11  7   7   THR THR A . n 
A 1 12  LEU 12  8   8   LEU LEU A . n 
A 1 13  GLY 13  9   9   GLY GLY A . n 
...
```


This secttion of a mmCIF file contains a mapping between the primary sequence and the cordinate data. Another useful piece of data with can extract from this is a mapping between the author chains codes and those assigned by PDB.

- asym_id represents the chain code as assigned by PDB
- mon_id represents the SEQRES (primary) sequence
- auth_seq_num represents the residue number as labeled in the cordinate data (with missing residues represented as a "?" )
- pdb_mon_id represents the corresponding residue in the cordinate data
- pdb_strand_id represents the chain code as assigned by the author
- pdb_ins_code represents the insertion code (This is sometimes used to preserve the sequence numbering when an insertion is present in the cordinate data but not the primary sequence)

With this mapping, you can now use the full primary sequence of protein structures but maintain a mapping back to the cordinate data. This is important as it allows us to maintain the full context sequence represented by a given protein structure.

 <!-- This is useful because you may build an alignment inlcluding the seqeuence of the PDB structure, and then you can use that structure as a structural representative for sites in the alignments. -->


<!-- Extracting the sequence that composes a PDB structure is not so simple though. The SEQRES header does not correspond 1:1 to the residues in the structure. You can extract the amino acids that build up the structure but now you may be left with a sequence that is missing regions because they were not solved (disordered) in the structure. If you are only extracted the sequence for use as a structural representative in an alignment, this may not be such a problem as usually the alignment algorithm will “figure it out” and add gaps in those missing regions. But if you intend to use the extracted sequence to benchmark predictors, the missing regions may be an issue, because the extracted sequence does not represent a true biological sequence due to the missing regions. -->