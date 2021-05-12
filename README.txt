These are the supplementary files for reproducing experiments described in the paper 
Vampire With a Brain Is a Good ITP Hammer (M. Suda) submitted to FroCoS 2021.

1) Where to find the Mizar bushy benchmark (in the TPTP format):
http://grid01.ciirc.cvut.cz/~mptp/7.13.01_4.181.1147/MPTP2/problems_small_consist.tar.gz

2) Where to get the used version of vampire:
vampire branch: https://github.com/vprover/vampire/tree/deepire3
commit: https://github.com/vprover/vampire/commit/110f414207d632819dea4cf01a1ddaca86d0cca3 

3) To compile vampire, libtorch from
https://download.pytorch.org/libtorch/cu102/libtorch-shared-with-deps-1.7.0.zip
should be unzipped under vampire/
If needed, please update the respective paths in vampire/Makefile accordingly.
Then run
make vampire_rel
to produce
vampire_rel_deepire3_4869

4) The baseline strategy V is encoded via
dis+1010_3:2_acc=on:afr=on:afp=1000:afq=1.2:amm=sco:bs=on:ccuc=first:fde=none:nm=0:nwc=4:urr=ec_only_100

5) We used a set of shell scripts for parallel execution on the benchmark problems, namely, these are the files
filenamify_path.sh
run_deepire3_def_10s.sh
run_deepire3_def_500s.sh
run_in_parallel_plus.sh
all.txt -- the list of all the files extracted from 1)
pepa-solved -- an auxiliary list that in the end did not play a role in the evaluation

6) The first run, to evaluate V, was executed using 
./run_in_parallel_plus.sh 30 all.txt ./run_deepire3_def_10s.sh ./vampire_rel_deepire3_4869 "--decode dis+1010_3:2_acc=on:afr=on:afp=1000:afq=1.2:amm=sco:bs=on:ccuc=first:fde=none:nm=0:nwc=4:urr=ec_only_100" existing_dir_to_save_results_to/mizar_deepire3_d4869_10s_base

7) To make vampire_deepire output log to train from, an option "-s4k on" was triggered and a larger time limit was imposed, since printing out the logs take extra time. Nevertheless, only logs for the problems solved at 6) were generated.
./run_in_parallel_plus.sh 30 base.solved.txt ./run_deepire3_def_500s.sh ./vampire_rel_deepire3_4869 "--decode dis+1010_3:2_acc=on:afr=on:afp=1000:afq=1.2:amm=sco:bs=on:ccuc=first:fde=none:nm=0:nwc=4:urr=ec_only_5000 -p on --proof_extra free --output_axiom_names on -s4k on" existing_dir_to_save_results_to/mizar_deepire3_d4869_10s_base_s4k-on

8) The training was done with a set of python scripts made publicly available via
https://github.com/quickbeam123/deepire3.1
commit: https://github.com/quickbeam123/deepire3.1/commit/af305223d15678fbdf54af734edadf26cc5e2111

9) The vampire run logs were first read by a log_loader script from the deepire3.1 repo:
(the folder structure rooted in ``strat1new_better'' along with the used hyperparams.py files are part of this repository)

cp strat1new_better/loop1/hyperparams.py .
./log_loader.py strat1new_better/loop0/ strat1new_better/loop0_logs.txt > strat1new_better/loop1/log_loading.txt

where loop0_logs.txt contained a line-by-line listing to access the files obtained in 7)

this left strat1new_better/loop0/ with (among others) the new files
raw_log_data_avF_thaxAxiomNames.pt
data_sign.pt

storing all the necessary info to continue the pipeline.

10) Three independent training-ready compilations for different ``revealed axiom set'' sizes m = 500,1000,2000 were initially produced

cp strat1new_better/l0_thax500/hyperparams.py .
./compressor.py strat1new_better/l0_thax500/ strat1new_better/loop0/raw_log_data_avF_thaxAxiomNames_useSineTrue.pt strat1new_better/loop0/data_sign.pt > strat1new_better/l0_thax500/compression.txt

cp strat1new_better/l0_thax1000/hyperparams.py .
./compressor.py strat1new_better/l0_thax1000/ strat1new_better/loop0/raw_log_data_avF_thaxAxiomNames_useSineTrue.pt strat1new_better/loop0/data_sign.pt > strat1new_better/l0_thax1000/compression.txt

cp strat1new_better/l0_thax2000/hyperparams.py .
./compressor.py strat1new_better/l0_thax2000/ strat1new_better/loop0/raw_log_data_avF_thaxAxiomNames_useSineTrue.pt strat1new_better/loop0/data_sign.pt > strat1new_better/l0_thax2000/compression.txt

11) The training was performed via calls (each requiring 20 CPU cores and several days to finish)

cp strat1new_better/l0_thax1000/run1-default/hyperparams.py .
./multi_inf_parallel_files_continuous.py strat1new_better/l0_thax1000 strat1new_better/l0_thax1000/run1-default/ 2>&1

cp strat1new_better/l0_thax1000/run5-64/hyperparams.py .
./multi_inf_parallel_files_continuous.py strat1new_better/l0_thax1000 strat1new_better/l0_thax1000/run5-64/ 2>&1

cp strat1new_better/l0_thax1000/run6-256/hyperparams.py .
./multi_inf_parallel_files_continuous.py strat1new_better/l0_thax1000 strat1new_better/l0_thax1000/run6-256/ 2>&1

cp strat1new_better/l0_thax500/run3/hyperparams.py .
./multi_inf_parallel_files_continuous.py strat1new_better/l0_thax500 strat1new_better/l0_thax500/run3/ 2>&1

cp strat1new_better/l0_thax2000/run4/hyperparams.py .
./multi_inf_parallel_files_continuous.py strat1new_better/l0_thax2000 strat1new_better/l0_thax2000/run4/ strat1new_better/l0_thax2000/run4/check-epoch43.pt 2>&1

12) To obtain validation losses (and thus to determine the best model) the validator.py script was used as in:

./validator.py strat1new_better/l0_thax1000/ strat1new_better/l0_thax1000/run1-default/ strat1new_better/l0_thax1000/run1-validate/

13) the models to be used later in vampire were extracted as in:

./exporter.py strat1new_better/l0_thax1000/data_sign.pt strat1new_better/l0_thax1000/run1-default/check-epoch60.pt strat1new_better/models/loop1_run1_i60_bestV.pt

Our actual models can be found under in this repo under:

strat1new_better/models/

14) To run and evaluate vampire under the guidance of a specific model, we executed commands like:

./run_in_parallel_plus.sh 30 all.txt ./run_deepire3_def_10s.sh ./vampire_rel_deepire3_4869 "--decode dis+1010_3:2_acc=on:afr=on:afp=1000:afq=1.2:amm=sco:bs=on:ccuc=first:fde=none:nm=0:nwc=4:urr=ec_only_100 -p off --output_axiom_names on -e4k ../mizar_strat1new/models/loop1_run1_i60_bestV.pt -nesq on -nesqr 2,1" existing_dir_to_save_results_to/mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1

15) Finally, to fetch and collect the results from the directories filled up with vampire's run logs, such as "existing_dir_to_save_results_to/mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1", we used the following two scripts

scan_and_store_neurspeed.py
analyze_results_loops.py

./scan_and_store_neurspeed.py all.txt existing_dir_to_save_results_to/mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1
...
...
saving to mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1.pkl
Eval time on average 40.1168336901 percent

-- this are the average times spent on evaluating the network reported in the paper.

and finally:

./analyze_results_loops.py *.pkl

obtaining an output such as:

Absolute ranking:
mizar_deepire3_d4869_10s_strat1_model_loop3_run12_i66_bestbestV_nesqr-2.1.pkl 28947 0
...
mizar_deepire3_d4862_10s_strat1_loop0.pkl 20197 -8750

By loops:
mizar_deepire3_d4862_10s_strat1_loop0.pkl 20197 100.0 20197 0
Loop 0 20197 100.0 added 20197 still missing 0 accum 20197
mizar_deepire3_d4869_10s_strat1_model_loop1_run4_i46_bestV_nesqr-2.1.pkl 26014 128.801307125 6277 460
mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1.pkl 25805 127.766499975 6129 521
mizar_deepire3_d4869_10s_strat1_model_loop1_run5_i52_bestV_nesqr-2.1.pkl 25484 126.177155023 5879 592
...
Loop 1 28997 143.570827351 added 8868 still missing 68 accum 29065
mizar_deepire3_d4869_10s_strat1_model_loop2_run8_i48.6_bestV_nesqr-2.1_nesqc--0.5.pkl 27348 94.0925511784 1289 3006
mizar_deepire3_d4869_10s_strat1_model_loop2_run8_i48.6_bestV_nesqr-10.1.pkl 26995 92.8780319972 1076 3146
mizar_deepire3_d4869_10s_strat1_model_loop2_run8_i48.6_bestV_nesqr-5.1.pkl 27570 94.8563564425 1076 2571
mizar_deepire3_d4869_10s_strat1_model_loop2_run7_i65_bestV_nesqr-2.1_nesqc--0.5.pkl 27579 94.8873215207 1052 2538
...
Loop 2 31814 109.45811113 added 2955 still missing 206 accum 32020
mizar_deepire3_d4869_10s_strat1_model_loop3_run12_i66_bestbestV_nesqr-2.1.pkl 28947 90.4028732042 193 3266
...

Greedy cover:
mizar_deepire3_d4869_10s_strat1_model_loop3_run12_i66_bestbestV_nesqr-2.1.pkl contributes 28947 total 28947 not known 79 uniques 80
mizar_deepire3_d4869_10s_strat1_model_loop2_run8_i48.6_bestV_nesqr-2.1_nesqc--0.5.pkl contributes 1285 total 27348 not known 86 uniques 166
mizar_deepire3_d4869_10s_strat1_model_loop3_run13_i61_bestV_nesqr-2.1.pkl contributes 602 total 28329 not known 70 uniques 67
...
Total 32531 not known 257

16) Here is a list of the additional models that appeared in the experiments:

To get A^0 from sect 4.4,
we continued as in
10) with strat1new_better/l0_thax0/
11) with strat1new_better/l0_thax0/run10-default
finally as in 13) to get models/loop1_run10_i74_bestV.pt

To run M_noAx we used:
./run_in_parallel_plus.sh 30 all.txt ./run_deepire3_def_10s.sh ./vampire_rel_deepire3_4869 "--decode dis+1010_3:2_acc=on:afr=on:afp=1000:afq=1.2:amm=sco:bs=on:ccuc=first:fde=none:nm=0:nwc=4:urr=ec_only_100 -p off -e4k ../deepire3.1/strat1new_better/models/loop1_run1_i60_bestV.pt -nesq on -nesqr 2,1" /local/home/sudamar2/strat1new_better/mizar_deepire3_d4869_10s_strat1_model_loop1_run1_i60_bestV_nesqr-2.1_output_axiom_names-off
(i.e., left out "--output_axiom_names on")

Model R from sect 4.4,
is from:
strat1new_better/l0_thax1000/run14-swapout
and
models/loop1_run14_i95_bestV.pt

Model R_defR
is
loop1_run14_i95_norules.pt
exported using a hacked branch of deepire3.0: new_abstracting_model_export

Model S^0 is from:
strat1new_better/l0_thax1000/run2-noSine
and
models/loop1_run2_i58_bestV.pt

Models M_l=<something>
are in
models/loop1_run1_i60_sine<something>.pt
see hyperparams.py's FAKE_CONST_SINE_LEVEL for how they were exported

models/ also contains the remaining models from looping, the best one (\mathcal{B} in the paper) being:
loop3_run12_i66_bestbestV.pt

DISCLAIMER:

The provided scripts are only research prototypes and may require some debugging to be used on a new computer. 
In particular, some paths might have been hard-coded and would need to be changed appropriately for the scripts to work properly.
