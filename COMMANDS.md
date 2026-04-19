# My PD Commands Cheat Sheet

## Daily Git routine
git add .
git commit -m "message"
git push origin main

## If push rejected
git pull origin main
git push origin main

## Complete chip design flow from scratch

# Step 1 - Create design folder
mkdir -p ~/OpenLane/designs/DESIGN_NAME/src

# Step 2 - Download RTL
cd ~/OpenLane/designs/DESIGN_NAME/src
wget https://raw.githubusercontent.com/REPO/DESIGN.v

# Step 3 - Create config.json
cat > ~/OpenLane/designs/DESIGN_NAME/config.json << 'EOF'
{
    "DESIGN_NAME": "DESIGN_NAME",
    "VERILOG_FILES": "dir::src/DESIGN.v",
    "CLOCK_PORT": "clk",
    "CLOCK_PERIOD": 20,
    "DESIGN_IS_CORE": true,
    "FP_SIZING": "absolute",
    "DIE_AREA": "0 0 1000 1000",
    "FP_CORE_UTIL": 40,
    "PL_TARGET_DENSITY": 0.40,
    "PDK": "sky130A",
    "STD_CELL_LIB": "sky130_fd_sc_hd",
    "MAX_FANOUT_CONSTRAINT": 16,
    "CTS_CLK_BUFFER_LIST": "sky130_fd_sc_hd__clkbuf_4 sky130_fd_sc_hd__clkbuf_8 sky130_fd_sc_hd__clkbuf_16"
}
EOF

# Step 4 - Delete old runs (if rerunning)
rm -rf ~/OpenLane/designs/DESIGN_NAME/runs

# Step 5 - Mount OpenLane container
cd ~/OpenLane
make mount

# Step 6 - Run full flow (inside container)
./flow.tcl -design DESIGN_NAME

# Step 7 - Exit container
exit

# Step 8 - Check results
tail -10 ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/openlane.log

# Step 9 - Count violations
grep "VIOLATED" ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/signoff/31-rcx_sta.checks.rpt | wc -l

# Step 10 - View timing report
cat ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/signoff/31-rcx_sta.checks.rpt | head -50

# Step 11 - View manufacturability report
cat ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/manufacturability.rpt

# Step 12 - Copy reports to portfolio
cp ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/manufacturability.rpt ~/DESIGN_NAME-sky130/
cp ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/metrics.csv ~/DESIGN_NAME-sky130/
cp ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/reports/signoff/31-rcx_sta.checks.rpt ~/DESIGN_NAME-sky130/timing_report.rpt

# Step 13 - Open layout in KLayout
klayout ~/OpenLane/designs/DESIGN_NAME/runs/RUN_*/results/final/gds/DESIGN_NAME.gds

## Check results
tail -10 ~/OpenLane/designs/picorv32/runs/RUN_*/openlane.log
grep "VIOLATED" ~/OpenLane/designs/picorv32/runs/RUN_*/reports/signoff/31-rcx_sta.checks.rpt | wc -l

## Open layout
klayout ~/OpenLane/designs/picorv32/runs/RUN_*/results/final/gds/picorv32.gds

## Take screenshot
gnome-screenshot -d 5 -f ~/picorv32-sky130/screenshot.png

## Copy files to portfolio
cp ~/OpenLane/designs/picorv32/runs/RUN_*/reports/manufacturability.rpt ~/picorv32-sky130/
cp ~/OpenLane/designs/picorv32/runs/RUN_*/reports/metrics.csv ~/picorv32-sky130/

## Understanding layout layers in SKY130
# Layer numbers in KLayout
64/20  → nwell (N-well region)
65/20  → diff (active diffusion)
66/20  → poly (polysilicon gate)
67/20  → licon (local interconnect contact)
68/20  → li1 (local interconnect metal)
69/20  → mcon (metal 1 contact)
70/20  → met1 (metal 1 - power rails)
71/20  → via (metal 1-2 via)
72/20  → met2 (metal 2 - routing)
73/20  → via2 (metal 2-3 via)
74/20  → met3 (metal 3 - routing)

## Docker commands

# Check if container running
docker ps

# Fix docker permission
sudo chmod 666 /var/run/docker.sock
