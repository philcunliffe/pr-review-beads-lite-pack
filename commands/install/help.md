Install the bundled PR review workflow formulas into this city's beads-lite stores.

Examples:

    gc pr-review install
    gc pr-review install --no-rigs

The command copies `formulas/*.formula.toml` into the city `.beads/formulas`
directory and, by default, every registered rig `.beads/formulas` directory.
Run it after first importing this pack, and again after updating formula files.

This pack is formula-only. Dispatch formulas to the worker sessions supplied by
`gastown-beads-lite-pack`, usually:

    gc sling <rig>/gastown-beads-lite.polecat mol-adopt-pr --formula --var pr=<url-or-number>
