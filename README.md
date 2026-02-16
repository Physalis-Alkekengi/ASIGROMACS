# **Protocol: Rational Design & Directed Evolution Pipeline for Polypropylene Degradation**

**Version:** 5.3

**Status:** Active

**Target:** Polypropylene (PP) Oxidation via Engineered GST/Peroxidase Scaffolds

**Compute Architecture:** CUDA-Accelerated GROMACS (Google Colab Environment)

## **1\. Overview**

This protocol defines the standard operating procedure (SOP) for the *in silico* evaluation of enzyme variants targeting the degradation of oxidized polypropylene oligomers. The pipeline integrates structural inference (AlphaFold/Protenix), topology regularization (RDKit/OpenBabel), system solvation (CHARMM-GUI), and high-performance molecular dynamics (GROMACS).

**Key Metrics:**

* **Stability:** Root Mean Square Deviation (RMSD) of the backbone.  
* **Affinity:** Non-bonded Interaction Energy (![][image1]).  
* **Catalytic Feasibility:** Geometric proximity of substrate to nucleophilic residues (NAC \< 3.5 Å).

## **2\. Prerequisites & Dependencies**

### **Computational Resources**

* **Runtime Environment:** Google Colab Pro/Pro+ (NVIDIA T4/A100 GPU required).  
* **Binary Engine:** MY\_GROMACS\_GPU.zip (Pre-compiled GROMACS 2023.x with CUDA support).  
* **Scripts:**  
  1. 01\_Topology\_Preprocessor.ipynb  
  2. 02\_HPC\_Simulation.ipynb  
  3. 03\_PostHoc\_Analysis.ipynb

### **Chemical Definitions**

* **Substrate:** Oxidized Polypropylene Trimer (4-methylheptan-2-one derivative).  
  * *SMILES:* CC(C)CC(=O)CC(C)C  
* **Cofactor:** L-Glutathione (Reduced, for GST coupling).  
  * *SMILES:* C(CC(=O)N\[C@@H\](CS)C(=O)NCC(=O)O)\[C@@H\](C(=O)O)N

## **3\. Workflow Execution**

### **Phase I: Structural Inference (Upstream)**

**Objective:** Generation of ternary complex coordinates (Enzyme \+ Cofactor \+ Substrate).

1. **Platform:** Protenix / AlphaFold 3 Server.  
2. **Input Configuration:**  
   * **Sequence:** Target Variant FASTA.  
   * **Ligands:** Explicit definition of PP and GSH.  
3. **Deliverable:** .cif structure file (e.g., wt\_GSTO1\_PP\_v3\_sample\_0.cif).  
   * *Note:* Ensure filename consistency with the TARGET\_MAP variable in Phase II.

### **Phase II: Topology Regularization**

**Objective:** Stereochemical correction and forcefield topology generation.

**Script:** 01\_Topology\_Preprocessor.ipynb

1. **Input:** Upload the Phase I .cif file.  
2. **Module Execution:**  
   * **Ligand Conformational Sampling:** Generates 50 conformers per ligand using ETKDGv3; minimizes energy via MMFF94.  
   * **Topology Alignment:** Aligns optimized ligands to the target crystal structure coordinates.  
   * **Standardization:** Renames atoms (C1, C2...) to ensure CHARMM forcefield compatibility.  
3. **Artifacts (Download Required):**  
   * final\_system\_v5\_3.pdb (Master Coordinate File)  
   * plastic\_final3.mol2 (Substrate Topology)  
   * gsh\_final3.mol2 (Cofactor Topology)

### **Phase III: System Construction (CHARMM-GUI)**

**Objective:** Solvation, ionization, and parameter assignment.

**Platform:** [CHARMM-GUI Solution Builder](https://www.charmm-gui.org/)

1. **Upload Configuration:**  
   * Input: final\_system\_v5\_3.pdb.  
   * **Hetero Chain Handling:** Select "Upload RTF/MOL2".  
     * Map LIG ![][image2] plastic\_final3.mol2.  
     * Map GSH ![][image2] gsh\_final3.mol2.  
2. **Parameterization:**  
   * Forcefield: **CHARMM36m** (Standard Protein/Lipid/Ligand).  
   * Ligand Reader: CGenFF (Verify penalty scores \< 50).  
3. **Simulation Box:**  
   * Geometry: Rectangular.  
   * Edge Distance: 10 Å (Minimum Image Convention buffer).  
   * Fit: "Fit to Protein".  
4. **Solvent Environment:**  
   * Ion Placement: Monte-Carlo.  
   * Ionic Strength: 0.15 M KCl (Physiological).  
5. **Output:** Download charmm-gui.tgz.

### **Phase IV: Production Dynamics (GPU)**

**Objective:** Thermodynamic equilibration and trajectory generation.

**Script:** 02\_HPC\_Simulation.ipynb

1. **Environment Initialization:**  
   * Upload MY\_GROMACS\_GPU.zip (Engine).  
   * Upload charmm-gui.tgz (System).  
2. **Pipeline Execution:**  
   * **Minimization:** Steepest Descent (converge to ![][image3] kJ/mol/nm).  
   * **Equilibration:** NVT/NPT ensembles with positional restraints.  
   * **Telemetry:** Monitor ns/day to verify GPU offloading.  
3. **Production Run:**  
   * Ensemble: NPT (303.15 K, 1 bar).  
   * Duration: 1 ns (Benchmarking) / 100 ns (Production).  
   * *Auto-Process:* trjconv removes solvent and centers the protein.  
4. **Data Archival:**  
   * Script generates FINAL\_SIMULATION\_DATA.zip.  
   * **Action:** Download immediately.

### **Phase V: Quantitative Auditing**

**Objective:** Geometric and energetic validation of the complex.

**Script:** 03\_PostHoc\_Analysis.ipynb

1. **Input:** Upload FINAL\_SIMULATION\_DATA.zip.  
2. **System Audit:**  
   * Parses .gro to classify residues.  
   * *Pass Criteria:* Detection of LIG residues with Carbon atoms.  
3. **Interaction Energy (![][image4]):**  
   * Recalculates short-range non-bonded energies (LJ \+ Coulomb).  
   * **Thresholds:**  
     * **![][image5]** kJ/mol ![][image2] **Dissociation (Fail)**.  
     * ![][image6] kJ/mol ![][image2] **Stable Complex (Pass)**.  
4. **Catalytic Proximity (NAC Analysis):**  
   * Measures minimal distance (![][image7]) between Substrate and Nucleophile (Cys/Ser/His/Tyr).  
   * **Classifications:**  
     * **Reactive (NAC):** ![][image8] Å.  
     * **Metastable:** ![][image9].  
     * **Non-Reactive:** ![][image10] Å.

## **4\. Software Architecture & References**

The validity of this *in silico* pipeline relies on the following open-source algorithms and web servers.

### **Core Engines**

* **GROMACS (2023.x):** High-performance molecular dynamics engine.  
  * *Citation:* Abraham, M. J., et al. (2015). *SoftwareX*, 1-2, 19-25.  
  * *URL:* [https://www.gromacs.org/](https://www.gromacs.org/)  
* **AlphaFold 3 / Protenix:** Deep learning-based protein structure prediction.  
  * *Citation:* Jumper, J., et al. (2021). *Nature*, 596, 583–589.

### **Topology & Cheminformatics**

* **CHARMM-GUI:** Automated simulation system builder and forcefield assignment.  
  * *Citation:* Jo, S., et al. (2008). *J. Comput. Chem.*, 29(11), 1859-1865.  
  * *URL:* [https://www.charmm-gui.org/](https://www.charmm-gui.org/)  
* **RDKit:** Open-source cheminformatics for conformer generation and SMILES parsing.  
  * *URL:* [https://www.rdkit.org/](https://www.rdkit.org/)  
* **Open Babel:** Chemical toolbox for file format interconversion.  
  * *Citation:* O'Boyle, N. M., et al. (2011). *J. Cheminform.*, 3, 33\.

### **Analysis Libraries**

* **MDTraj:** Trajectory analysis and geometric calculations.  
  * *Citation:* McGibbon, R. T., et al. (2015). *Biophys. J.*, 109(8), 1528-1532.  
* **CGenFF:** CHARMM General Force Field for organic ligands.  
  * *Citation:* Vanommeslaeghe, K., et al. (2010). *J. Comput. Chem.*, 31(4), 671-690.

[image1]: <data:image/png;base64,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>

[image2]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAAYCAYAAAAYl8YPAAAAqklEQVR4XmNgGAWjYBACKSkpWXl5+W4FBQUOdDmygJycXDkIo4uTBZSVlcWArtuvqKhohi5HFgAZBDTwiIyMjAqKhKioKA9QQpIMHAz07iOggZxww4CBWQESJBUDDXsGxP+B+uOR3EY6EBcX5wYashBoWB+6HEkA6CpXoCGrUbxHJmABuQiIPdAlSAZA10gDXbUZmHhF0OVIBsbGxqxAA4WATEZ0uVEwwAAARTIta6WFRLIAAAAASUVORK5CYII=>

[image3]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGkAAAAZCAYAAAAyoAD7AAAEnUlEQVR4Xu1ZXYhNURS+t6HI72C65u/se2+jaRByQyZ5MQ88eJEyNeVB+XkQRVFCHnggNE0exAgPEg0leaApohAPUialUWaSoiRTHiTG95299tizzD3jzvVw6Hy12mevtfY656xv/6x7biqVIEGCBAkSxBDZbHaRMebMnwh8p+vx/xuCIMijSWu9Qz6fD5CL5qhcVFVVTa6vr59P0TYfjFFXVzenUCiM17ZhyOVyBg+2HnIQNx+EdEjfyR7ontHW2Ng4RY//X8BE4V1X4T3vMcnaDqSR1A20Q87B9xX697UT8wcZgO2KyFEQMdH3gW42fK5D3iPODbQfIdthqvD9foMEv4UAE7SNQLC9WhdzVGAmL9FKDRKCiZoRktaORBJWzzToH0Oe+nr4tyBfh1wf9uWQI56Ly+tX2tiHf6uxi6HN+TDnzD3k7a+RCrhZJR8giogoW5yQyWQm4Vl3Qvohp7U9CsVIIhnQ/4B0+Xr0myRvldInIUPJJ0DAViElJE/I+IYxK3w/GTvo64YBM2khHAZGGDg0K3B9wLfFEXjGash7yP7a2tqZ2j4aIkjaywQi4Rd9vdxvgPnjGI5lDN9HYg5CujmB0PZBvkBfUH7hPXzdMMDYRgfsnbVOx32UM8j3iynSeM65xu7xr/X+XwqKkURyIkj6ISuN130RJPXU1NTMIkFRJI143MjAHjoEdovox/WHogNiAJ4f3NshdzCLF6QiqrFSMEaSmDeOG40kriD6RJKk7+2MBRn4zVOn0b/k9WOBhoaGqYGtNrmlndP2chFbkryDrcdTj0P/stePBbK2BP4MOcsX1vZyEVeSSEaX3Oi80oclY9zAMwfPtgnyDltdo7aXg2IkuQQWISlM+B8UDmFcU2rhwEIBhjcSZFjp+A+gwtiz9BGkmX3tUCqKkQTdGsnRLV8vVXFYEIhfJ3cm3we6XTK2Q/rdRooN5ddJP1/nDMckwAsEz2q7A+w5BN2B9ivaPVCl+QMP17dB9Az6cCa4l+NXDOA5r/lpBNefIOtQFtehPcyD39hZuN3YlwjLWO+WJYPPaOyXgJ0sdbV9NEgxsg0xHvE5tR363ZDvri8/fs9D5jmdrJTXfi7Zh9x0Vafk4CVc7jOH1OF6mbGrq92NG/pdZCxBvvAcGjfkKECQRRTjzTIEzEO32vkYO1M4liVxO33Fb4WxS7zJxRH/Jn4RQFuN51nKcS5WOUC8/cYWFicZW9s1ZPW499dyz/kJiUehO4K2NbCfcz54oUJA9xDSC/sWCq674T/b95Fc9qJ9Yuwk/UTCxzK5NHhOhUuWQNAWN+MC+WLBa28L3cd+1hYmD/R3P+jWuW3ib0N+4230k/y3gLjrEbcNE2tlkQ+jFYGtmNtIZqrI5ONYxmA85KFe28cEBMwwsa5v7IwKf0eRMPT74LMAbbOxq3RN6ldh0kFC+GBCDM8TEs4XSOdsAfDbCk5QIrgi3FaHBFeDmFfOhuvN0N2FLEaXv7F41lyDnIJcgv0C2hP0Q5yrxm5FJJlnwHG3NycoE2ppp/X2lVX/r3gVEs8ofoAMqy/6uRWoxyRIkCBBggQJ4oefhjKkix009LsAAAAASUVORK5CYII=>

[image4]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACEAAAAZCAYAAAC/zUevAAACH0lEQVR4Xu2VT0iUURTFRzIoNMJqHJx/b/5Eg1AWTi4Mt4Ju2igRuGkXQa3btolWgqibwk2L0EURgQq5EnTntjaC4GwFN+5cRP1Oc5+9HjRNOrNqDhzed+9537v3u/e+mUSigw4awDk3Cd80w/jdliGbzd7K5/PT8BOBaoVC4ZHZ09gz2K9ZD+D3+N2WgoB9BNkh4ONYE5LJZC/6euxvKQgwCA9Jphr60+n0Na2WxMdQazlUdpVbFfE+Czyo53K53I+28uuNNoBg89bzrsA3FSbVVvhWwONgIBfgt3hv2+BbAXe8L5PJXCWZz95mYC+oPd4uFotD6M+q1ep57zsTCL5kSSx5HwFK2MtmdpPEWzjhde3F3ggTawT2z4bn/wZ/NS2JmVgXcrncTbTVUql0Odaawd+uv8p6mw1Hop5jHXRbpZ7GQrNw9Znbh6Ox9hMIH1QFu37nzN2lmcB/zyq0KJ8EvuYFyeJye3BSPip0g+f3nLErjfUB9qZvlavfvHf+jBMoK3hsQRpRe06+gCTuwIf4vhIwpYHleQB+oW337eyXcE2a2X9uxSmh9iy7oDp8+Rh2zdkPG+smvuf+Bew9Ehzx9pmhQAqowPzxXYdX9JX4tiqVyiXbU9MwU6mKhllV088/ey/qasdn/jMIPs6h26xzBLorX9gKs7dd/UoOe5v9r0j2SSKei9NCA+f7LaRSqZ5EcHisy27296SD/xs/AFg7lqrJ//XjAAAAAElFTkSuQmCC>

[image5]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAE0AAAAZCAYAAAB0FqNRAAADjklEQVR4Xu1YS2gTURRNaBTFPxqC+cwksRKMoIugiyLiD1FExB8tFMSdG6GgtEWpIrhyocsihSIKbkREEVHUhYigUJAurAtBaF0IKlnowoXSxnOS99LXm0wySaa2lDlwmMy5n8w7M+/NSwIBHz7+KzKZzAqptQLVr03qCwqWZT0Mh8PLpd4s2A/MSX1Bwcm0ZDK5xLbtXnDIBXuZr/vNoWmhaDS6Toq4vvVSc0Iul1uE/A6MZ7WMleFkGosR24sGI2AB/IzzU+AJEk27oN1QsfsoCbFuLk2jOeAE+NK4oe9wPZdlbhUEMaZO5OfBYdR8wvlrMCkTHU3TUE2mkLdPxgj0vI1Yvz6fDdMSicS2SCSyTOoSyrSPyqxBXMdJN3XpdHoVzQVHULNG6xwzx27m6kA90wrgeDwej2kNNVk9DWga4seMmGemccDo1QPe5MBkXEKZNir1etDm2MaMIXC+CcybRhbh0rQnes2CeUvx+R7vvopfT6VS23W+F6apwXPqfwMHZNwJLZjWz3HyATB11W8C49tq6jVNo8NsxqZaQ+MD0N5XW3CJFk0L8im21frJGyQTakGbBh4CB0n0OSzzJNRscTSNT6Kp1zQNTc6wCMfT1vTiPwl2y1yNRk1TPcdxfI47ugVSUOa4BfeI6PXC1HC+m72dxki4MG2m8TVM412/haK7/KxFnI+apsi1xq1p7e3tK5HXh37DYErGvYIa+BRniIxpeGYapx8Kxvi0mTq0x6ZRiF8xwq5NS5Ze7z95YTLmJdTTVwAvypiGZ6ZB34GCv7UMwHSKIL7Z1NyaRnDNwnd8BR+gV0bGG0QIfa6CeVPk2KoZYsKq8yKoGE8N04qNLPm6nQY3gxd4NMVGTFNoQ58j+K4x8C3YIRPcQA+Q15zNZhdrnTeWGnjOzDeB2EGVU94lEHxrQvtV8dKrYlpQJfMJKBh6McZBgXcYQ+0zEW/GtArYpR35F7DHzeZUA3XHwaf6XP0kGkKfvsDMdZkGTcKgXYZ2XmmdutYqrek/dE4Z0jTV0A25GSxvajW8MI2IxWJr0X/ALu3V3K57vKln1RrFXwUfwD8B8a+LXVpL35h9aRLqrkH7jWMXxvAIn7+De8zaIqRprcIr0zTUuvcKx40y5gTkHwW7sQHfL9/u9YCpmGAtZttOGinjRcx30+YlfNOagG9aE8D8vWS+ZlsF+2Et2SB1Hz58+JhF/ANnRDUh2KWXxgAAAABJRU5ErkJggg==>

[image6]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFYAAAAZCAYAAACrWNlOAAAEKElEQVR4Xu1XXUhUQRRe0aKoiCiz/Nm7rpRoYJFUGEL0SxHSn0EgWG9FCIFgUlQEPddDkA8mhEIIEVGEFNVDRNCPkAZZDyXoiz2E9GRE4W7ft3dGj4e97nV3VdL94HDvfOfMmZlvZu7MDQQyyOC/RGlp6TLNpQHZmph3CAaDD3Nzc5dqPlkwF3JWan7ewUvYUCi0yHGcJlirD2tiPOvNsrA5+fn5qzSJ/q3VHJBNHlZVUlKyWjsTAfU2FxYWrqusrFygfTF4CcsK8O1Ggm5YFNaPcj2slgYhT4C7YXz3USWH9WZTWCPUIOyFMz7pb9GfKzIOfd8B/iusx8T0FBUVbZExXkDdNYh/gGc78j7C+zCsQcd5CmthKkYQt0f7CNNAsy37FbagoKAQq6tI86nACPvZiNWCfhzPy8tbosJy4PsOX1iSZpyd9EtewiymKKxOcNzZXbBiGetHWCYawLIvsBzqlNstR2HhP2p9foUtLi7eiHqvEfsukKbDznGF7dW8BPxlsG60u0LxfU4cwS2EgH8RUy194G46QuwYfArbxcQsQ+DFeL9ntw581yHSVhvvV1iDLE6SYz4zzK0DpgI/wqKdGsS81GMmZ8Z6QPIWJjc/MyN6fCg3g2+T3KTCclbZGCtaDqLuB/ch3iFBTFHYGEynbznudryk/X5h8vTCDsJaaBRSxhgRPIXF+E5L3oJjoqg0PT6Tc2zxWdJTWDaCCoN4ngqOH1ijjl72AskIq2HaGcDzGXZDhfZ7gXdy1HsuOZR3Mo8dYyJh5SKS8CHsxJyTCMttegcV7vLdkij3ysThcHi5fSfSIawBr0O7IMp7PlnWAX7guKs4wp3GclwR3LiZEZZbHcF9emuAeyzFhP+qcKdTWCIL7VXBPuKbXqKdfmBWcRR2keW4IgRmUFjw1Y57AnqKhG2aB/8GyaVJ2GxM2CFOLOxNQOyYScBr1DXYsCTZHwqGfO0sBxMcXvRL3sJJcHjZ/JL0EpazwIYmXEsEspDsAp+STFVY3jvR7hCsAxNXqv1eEAOPlpeXL7Q8J58crNGUec3r04cvuH7HPTzLJG9h+sUfj0hQ3enBtemdHU/YLNM4BxcVfMwHbjusgz7Ufar8SQkbdL/lQ3iei3Oh9w3kOAZ7Ysv8e0S5FXnPByaeE6NqUXBco4jbK2IoYhT2A5++9eT4U4PyF9R9ZePwvg3cCHNYLgYtrEnmxyKO+DGwmIKwnMAKntiww57/3FMDBWrgtnTcv69PsD8BdfCBOwv7BbuNvtaaZz3r2xiuQPC/YZ1SH/CbwH2D1cEaYT+5MKx/DFrYVOFXWHTwDFc8Z1z7UgUGe4QDx4G3T99aLLD6VqL9GrR/knHaPxnMTqjjpHj+ls+WsHMeGWGnCRlhpwn4VlwOyX/cFMFcyV7mM8gggwzmCP4Bi1hncSYsxLMAAAAASUVORK5CYII=>

[image7]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACQAAAAZCAYAAABZ5IzrAAACRUlEQVR4Xu2VPWhUQRSFX0giMSoRdV2zf7P7tlhY/MNFQVCwsBAkjY1ITB2rFCm0ExtBECwUG5tgrZ0EIghGLAxoaVDEFGsj2gQEhSRg/I5vJkyGbDCJb9PsgcPM/Xl7z9w7mURRBx2khFKpNAK/iMaYxTDedhQKhSOIuYqYj3ApjG8LEHQeMb/h0zC2LUDQDcQsw/Ew1k50V6vVg+VyuQ8hkxoXws6ESakjm83uovAdBHyD0/AR/A5nc7ncgTA/VcRxPEDhhWKxeNb5sB9qXHRq1M9tCyh6TePRmJwP+wX8hciTfu5GkM/nC3T9ug4cxlrCdmcm7AS+r1sdF9+Pww8SFsZaolKpHOOjH1p9v8bF6SbYdvn+1EHRBsV/hp2QIDjMtofuXfFjqYIXeSeFn7Hmrb0P+wlswkHdL0TfhGO69PgW3L1g/4b9c31j36zpTCaz28YGZRv7hunpYH8bzjONU6yn4XvlrVYUJZdPQYo/Zv3Metkk788r1tf8gMF3XD/uihKL2TfxX9BvsL9vkhddHS3bgk0OcVhxTQB7UsJt/rBZ744iaj+Fs41Go1e2ivo26FFBFZZRSv6tzLkLy/6t/4fB/hY5U5qAzf97AH1n7Qkx2uwdlTgJgJdkm6T9K0+FLRaTd9Reg1l1QVdBXOMAc+ouHSqyHvJr/RN0emPHJZsCn/Cdc3FiL+EDeMLaM/AeeUPWXhmXtd/Bu/Ci820IGl29Xt/h7Fqttify2o24va5b1u5z4gU60c/S3SreQQf/C38ATpmXnG/VzmsAAAAASUVORK5CYII=>

[image8]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFYAAAAZCAYAAACrWNlOAAAEWklEQVR4Xu2XW4hVZRTH92G0Ei3DGg/O5exzZqYGRx2lY4mkGENGFy2QoELThwGN8AID5ouKL4IQWkz1EkIoRA8NQxDzkAw0MYGmvggJkgnZS6j4YFR4wcvvP3t9h32+uZzZzdAcpv2Hxfety3db3/rWXjsIUqRIkSLFf45cLvcu9LsoDMPbvn66oFgsziwUClnOuKC+vv4JXz8aNIamxhOLz3iycjQ0NCzBqZtY8AJ0x9dPB3Cu7dAVzvlNPp8/Rv8G/XVBBefU1tbOwXZAAQd9btQP/cU8j/j2w8AiL2J8D+rxddUMgqKFiAp9eRzZbHY257oPnWpqapormTlMsr6xHOQca5chp3YmiXY5do8t1OXrqhHss4M9/yTCse2+Pg45bqKOZZ31vm4s1DQ3N8+3hfugO0ywyjeqJhChs9jjZvbaS9sWVHjKDpxxTUtLy2OOZ556c+z2uJ2PRI7V08DwEAOuaFAYhfhV6HxdXd2Tvn01QM+P/e21PR/x9UmBo99jnn4XwaPBObaxsfF12m75St8j+bDMUBOhvIXhaieD/wy6z2Lb4rZTjIyeOHs6wd5+0xfdN0gK5lmIU96kfV9BRP9532YEZLA9ju0HTkC0P4XsKpfdULKymyrLK7o56B+c/WzJMCHkBBbfMRkOADXs5yx7PE3bId43mCjsBdwNo1QwrnTiEKsUojRi0XrKj0xkf+gGJ5IGGH9U0aVFfV1ScMHNzHcGWhkkPHQShFGOvUZALPJ1FaAo/lKkvqJqKcyfauNWWoDJvxgyqhJwSSvY10ldOP03fH1C1NhHuSzyzbFjpkDGbcXmOrQ3Ls9H5dfAUCBhVIT5249MW2Aj3RkMeCeum2oQBK3srRfqHPbBGCc409t2xs64vJJjY09eduec3Opipc+jQwKVKjDfqtQwfh7819BlaIHyL87fD+3Uxw3ZLSVtSyEn6X+nMbmo5h26LdoC4w7QXoJe0bwaT7+H9peCFe/O3m3u38CqmV256Ne74OtHg34iGPMR5dbDTqYyM4wcdjiwlxpGzpLsGmd+WjL2/xz8j5rDhmbkE9mVfU/0JUP4s4Xyr7RvhVH9+gPtoByBbBn9LucMdE30LyN/2TagskN/aIrwZWZ/nrHZfFQTv6o1VKK4dcMKhXgS6EBaj/lOQCuCcaQw7C+G0S/7EWifzgPtjjtHkYvsJvRVPAjgXwujMk9l6WAY/aF+6PQl6IsoJ7hJNUmcBzMY2AN1i8lFv7uXXHlB/4z3fGT/aRA7oG18ofp2OXtK1pODjJzKGn3MvclXjgDl2WI+SgsbufSXfIOxoFfL2C3MsV6+8vXjggbKkdAG8bQHdQAXcXKaohi7dksvqg9X6bkoVZjNYGtr66PWX8lBFitfVirIpzXsSZRyonIlshecHt330CfQM6ZfC32M85aL18cxngbsr+Wws//fQimhra3tIcdb5JWeOU5+3M+XPh94eW8EfYoUKVKkSFERDwB+ri1AGUIh7gAAAABJRU5ErkJggg==>

[image9]: <data:image/png;base64,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>

[image10]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFYAAAAZCAYAAACrWNlOAAAEGklEQVR4Xu1YTUhUURQe0aLoF80GnXHuzDg0OJVJ0w+GQYuCCmrRJsNq4aLaVGCQBBJthDKKsNqUEa0ikjbhohAqhIxq0UIpQhcGFRYRBkkmZN83777xeppp3jigou+Dj3vv+Xnv3nPPO/fOeDwuXLhw4WLKEQgEDoEfSKXUb6mfDYhEIkulLBwOL0slT4eysrI1ZDAYXCB1KeH3+9ciqAcR1HfgqNTPBmBdY+BHBOUe2hto76AdAqulrYTX610Eu1b6av9vaC9Iu5RAYLfD4Q/YLnVTDS4ETb6U5wKsqw+8yqCCLQjMZmmTCgw8OAI2C3mrk01hYBuVtasNUjfVwBdUqEvTSR3knIF1PS0uLl4s5ZmgA8i41JlybMxRGWwT+eXl5StZM2DUAY5iMTXSaDrg8/mKMJ8mcBC8LPXZYjKBpT39GFjEZY+p4xjyzgkbzwEU5/Wk6cjP4wvYW1pausLwn3YgexdiXvXgA8w5BlGetHECrhPP8vEr0Ott8WR4FmxKwIH/BHY8XjwJIRjB6bbVNsL4Op2Z3knPmYm8UChUiXk+Bmvj8fg8aZAOsH8JX2WI8rhm8JQhmwAHgaWuJCHAC45h0MHP3zbCuBMcRrA3Jj2zBBeMlx3PZrG5APO9BQ7mUod1YN9i7l6pIxwHVmfrC5mZkH1WOZYB+Lcxk7KtY5OFXvRNcAjv3S/1TgDfYTJdQjkOLHZmHTo/2JpG2vG2J0PNmSnA/KOY8yewnjVY6iV0gN4gsVYJ+c9UQbPh4PCyDkQM4nyYzEw6Kus6UYDdP2DqZhDyMcctYDf4mmNpkA46CFxjvSnXsn8SzQT0bbRL8ZU3gK2JgT5hH/J01ONCjO8rndKsv5jEWfAEDzfIRtA/rUtIN/qP9B2Td97EbqENwe8c2n5wF59Lf/Tb0b63DwzbPjkzB9C3F57ivL2EpN4pWPcxxzMeYzM4L+A72k22TFlnzRj41c5uzpkljuuz7TAOcjzha8H90A9hT9D6SdfH+qSs++sztF18IWRVytqRRDCgC6M/APlOPkNZl2b+QmOGV2n7Xh4CQetOvJvvwObstd+rxIGZCbC/qKyANvFOK/XZQlkJ1KOMn7OytgatS/8v8K6ZBDpmzxGHI6SyfsV1mr62YRGDYJ/gfIg5Bgrg2K50qgesn7v9fAHH6L/iJLStbX/NY9RoZX0FFezrzWlMWmcAbFeDh53UzyzAUlIB1mHutQjqBmmQAfRP+Opszv48YpAZSHAfx2iblZFxDBqzGHaVurxUYFyDfoSlQtt0RaPRJbpfzX+FeOiwrIy/aY5BfxLJmshaCdk2Ww/dE2X9obFe63eAV+ws4OFolgH2YXvJtp+zYEmIxWLz7bHOvGTqI8jLZb2UY4/4VFLoXbhw4cKFi4z4C6Z9MmXHSZW5AAAAAElFTkSuQmCC>