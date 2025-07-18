fhm-simplifier v0.1.0

A python package using the dask package to load, sanitize and output Franchise Hockey Manager CSV exports. Can output resulting files to .csv or .parquet. Files are meant to be used for further data analysis.

The program is strict with the CSVs so probably will not work with previous years or possibly even previous updates of the same game if they added or removed columns. Until it reaches full release.

Python version Python >= 3.13.5.
Python libraries used:
dask >= 2025.5.1
pandas >= 2.3.1
numpy >= 2.3.1
pyarrow >= 20.0.0

Dask is used to load pandas dataframes in chunks to ensure the computer does not run out of memory since all files are loaded at once and exported as the programs runs.

Follow instructions on dask website.
https://docs.dask.org/en/stable/install.html

Installed through pip using:
python -m pip install "dask[dataframe]"

Some issues with the games CSV files are:
1. team_lines.csv has 4 extra null values at the end of each row. Column labels are also not correct. For example after column 'PK3on5 L2 LD' is another 'PK3on5 L2 LD' label when the second one is supposed to be for the RD. That is why every CSV imported has column labels applied instead of using existing ones.
2. staff_ratings.csv has a ';' at the end of the first line indicating a column break but not a line break. index_col=False added and there is an extra column Between 'Trainer_Skill' and 'Evaluate_Abilities'. Added column named 'Blank' and then omitted it from following use_cols=[] statement. In addition there are ~700 repeat rows of 'StaffIds' added to end of file. Used first occurance of 'StaffId' as they seemed to be the correct ones.
3. staff_master.csv also has ~700 repeat rows of 'StaffIds' added to end of file. Also used first occurance.
4. goalie stats related CSVs are imported like all other CSVs as strings because 'SV_Per' for a goalie without a shot are stored as '00nan'. When trying to convert into Int32 causes error. 'Save_Per' with '00nan' converted to '0' first.
5. player_ratings.csv has a few values that are I suspect not used anymore that still get values. Ordered columns by how they show up on players and goalies and moved unused to end of row.

General improvements are:
1. Consistent stats naming conventions between files e.g TK, TkA.
2. Consistent dtypes.
3. Elaborating on hard to understand stat naming conventions e.g InD to Injury_Days or too short columns that could be confused e.g 'C' to 'Center'.
4. Shortening on common stats e.g Shot Block to SB.
5. No spaces between column names. Anything with a space was either removed e.g 'Staff Id' to 'StaffId' or adding an underscore 'Goalie Stamina' to Goalie_Stamina'.
6. Changed order of columns for either better grouping of relative stats or to be consistent with in-game displayed stats.
7. player_ratings.csv ratings only related to goalies had 'Goalie_' added as a prefix e.g 'Mental Toughness' to 'Goalie_Mental_Toughness'.
8. Consolidation of CSV files to add more context to each output file. 29 files imported. 22 files exported. There are 41 files exported from the games, but files related to career or retired stats were ignored (12 files).
9. Added 'Season' column to almost every exported file. Exceptions were files with GameIds (5 files: skater_stats_game, goalie_stats_game, games_penalties, games_result, games_scores).
10. Removed 'FranchiseId' column as was not sure its used anymore.
11. Ability to specify which leagues are included in data. e.g settings leagues to '0' will only leave NHL stats.

Default leagues (9) are:
1. '0' or 'NHL
2. '1' or 'AHL'
3. '2' or 'KHL'
4. '3' or 'SML'
5. '4' or 'SHL'
6. '10' or 'ECHL'
7. '11' or 'OHL'
8. '12' or 'WHL'
9. '13' or 'QMJHL'

Leagues can be added to line 82:
leagues = ['0', '1', '2', '3', '4', '10', '11', '12', '13']

Filepath to game CSV can be modified on line 37:
filepath = '/path/to/fhm11/saved_games/savegame.lg/import_export/csv/'

Output folder can be modified on line 95:
outfilepath = 'simplifiedCSV/'

Default export file type is parquet can be modified by changing df.to_parquet({fileName}, name_function='', write_index=False) to to_csv({fileName}, index=False) on lines:
1. 117        
2. 136       
3. 147
4. 158 
5. 169
6. 181
7. 205
8. 216
9. 227
10. 238
11. 249
12. 260
13. 280
14. 298
15. 308
16. 318
17. 335
18. 351
19. 372
20. 392
21. 403

or just find and replace.

Creating a list from team_data.csv that will be passed through and filter out unwanted data from CSVs.

Intended use is for SQL type data analysis so redundancy between files was a consideration.

For example team_lines are stored as PlayerId to be understood which player is which one would have to import data from players and compare PlayerId to Last_Name.

Examples can be found in example folder for intended use.

For some reason when using on files exported from multiplayer leagues the Ability and Potential in the exported CSVs were close but not exact even when using commissioner mode and turning on export with real values.
e.g Mitch Marner in game value showing Ability: 4.5 and Potential: 4.5. Exported CSVs showing Ability: 4.0 and Potential: 4.0

<img width="1887" height="949" alt="players2-Mitch_Marner" src="https://github.com/user-attachments/assets/3f384218-8c66-42f3-91c4-38f0110d5364" />

team_stats_playoffs.csv has no attendance information in relevant columns.

Simplified files still rely on exported game information being accurate.

The program does export 22 files, see attached photo for equivalent in-game information:
1. league - Consolidates league_data.csv, divisions.csv and conferences.csv.<br>
<img width="623" height="919" alt="league-NHL" src="https://github.com/user-attachments/assets/4d0aac2a-197e-4adf-ab72-304db84558a1" /><br>
2. teams - cleans up team_data.csv<br>
<img width="621" height="953" alt="teams-Toronto_Maple_Leafs" src="https://github.com/user-attachments/assets/5aba64a3-dead-4da6-ab65-95ec850d6a94" /><br>
3. players - consolidates player_master.csv and player_ratings.csv<br>
<img width="740" height="712" alt="players1-Mitch_Marner" src="https://github.com/user-attachments/assets/1d7a6956-cca5-4af1-b439-c815dc4aa026" /><br>
4. contract - cleans up player_contract.csv<br>
<img width="1886" height="574" alt="contract-Mitch_Marner" src="https://github.com/user-attachments/assets/85621856-5f19-437c-aef5-7b13de26a870" /><br>
5. contract_renewed - cleans up player_contract_renewed.csv<br>
<img width="1385" height="315" alt="contract_renewed-Mitch_Marner" src="https://github.com/user-attachments/assets/78ee6054-620b-4183-9fa3-de8814dab1f5" /><br>
6. player_rights - cleans up player_rights.csv<br>
<img width="697" height="193" alt="player_rights-Mitch_Marner" src="https://github.com/user-attachments/assets/1366d364-ecc5-4708-9314-d2260c12b011" /><br>
7. skater_stats_game - consolidates boxscore_skater_summary.csv and schedules.csv<br>
<img width="1850" height="384" alt="skater_stats_game2-Mikael_Backlund" src="https://github.com/user-attachments/assets/2717d6a7-fcac-4158-b4fb-98d83c0f1310" /><br>
8. skater_stats_rs - cleans up player_skater_stats_rs.csv<br>
<img width="1867" height="286" alt="skater_stats_rs-Mikael_Backlund" src="https://github.com/user-attachments/assets/9410d9ab-4daa-49a9-aa7c-b19315ac7829" /><br>
9. skater_stats_po - cleans up player_skater_stats_po.csv<br>
<img width="1868" height="286" alt="skater_stats_po-Mikael_Backlund" src="https://github.com/user-attachments/assets/087d0535-2f34-49ea-9d0f-41eeb1535773" /><br>
10. skater_stats_po - cleans up player_skater_stats_ps.csv<br>
11. goalie_stats_game - consolidates boxscore_goalie_summary.csv and schedules.csv<br>
<img width="1861" height="263" alt="goalie_stats_game-Dustin_Wolf" src="https://github.com/user-attachments/assets/e460a012-c75e-4660-89c1-e726541aa13c" /><br>
12. goalie_stats_rs - cleans up player_goalie_stats_rs.csv<br>
<img width="1864" height="283" alt="goalie_stats_rs-Dustin_Wolf" src="https://github.com/user-attachments/assets/59369c51-4f5e-42ae-ac32-78a1d67ab3eb" /><br>
13. goalie_stats_po - cleans up player_goalie_stats_po.csv<br>
<img width="1867" height="284" alt="goalie_stats_po-Dustin_Wolf" src="https://github.com/user-attachments/assets/683e7430-3c3a-420b-8698-2cfacbd514f8" /><br>
14. goalie_stats_ps - cleans up player_goalie_stats_po.csv<br>
15. draft - consolidates draft_info.csv and draft_index.csv<br>
<img width="1862" height="177" alt="draft-Mitch_Marner" src="https://github.com/user-attachments/assets/fa6ba898-fafa-49b9-b832-b70c742d99c6" /><br>
16. staff - consolidates staff_master.csv and staff_ratings.csv<br>
<img width="1920" height="1080" alt="staff-Craig_Conroy" src="https://github.com/user-attachments/assets/d6fe812e-9ef2-4562-901a-718fc021696b" /><br>
17. team_lines - cleans up team_lines.csv<br>
<img width="1920" height="1080" alt="team_lines-Toronto_Maple_Leafs" src="https://github.com/user-attachments/assets/280ff0d7-1940-4350-87c3-0ba57b1ef2da" /><br>
18. team_stats - consolidates team_stats.csv and team_records.csv<br>
<img width="1866" height="361" alt="team_stats-Calgary_Flames" src="https://github.com/user-attachments/assets/ad606c3b-1b6e-4b16-b24e-30b22b62063f" /><br>
19. team_playoff_stats - cleans up team_stats_playoffs.csv<br>
20. games_penalties - cleans up boxscore_period_penalties_summary.csv<br>
21. games_result - cleans up boxscore_summary.csv<br>
22. games_scores - boxscore_period_scoring_summary.csv<br>
