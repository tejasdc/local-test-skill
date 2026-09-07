# Evidence for testing rendered UI behavior

Source: tejas.nyc mobile homepage incident, Slack thread `1788813226.497669` in `C0BNLJ6Q68M`, investigated 2026-09-07.

The original quote change (`tejasdc/chann.app` commit `b3d089d9256fa13cb74673137070bc4e3893aa06`) included a Playwright WebKit check asserting that the mobile hero field was exactly 497px tall. The subsequent spacing change (`1209477`) changed that assertion to 473px. The fixed outer height left a large blank area on a phone even though the quote was contained and the test passed. Independent reviews caught clipping but accepted the containment correction.

The correction (`038aa7c`) restored normal mobile document flow and added checks for quote-to-wave spacing, the introduction remaining in the first screen, marker proximity, and compact navigation. A later user-requested mobile format (`9b6650e`) added single-line text-range and centering checks.

This is evidence for testing the rendered requirement and inspecting the composition. It does not establish that the old `local-test` instruction caused the original implementation; it establishes that “never test CSS” is incompatible with verifying this class of user-visible failure.
