#!/usr/bin/env python3
import sys, json, re

"""Update README.md: fix TODO list to reflect main→dev changes:
  * Marked 11 TODO items as done (removed from code in dev):
    - luffy/deepscaler/utils.py:45, 46, 47, 48, 49 (API client init / auth / backoff / error handling / response parsing)
    - luffy/verl/verl/protocol.py:114, 115, 116 (fold batch dim TODOs)
    - luffy/verl/verl/protocol.py:131, 132, 134 (unfold batch dim TODOs)
  * Updated line numbers for 18 TODO items whose locations shifted in dev.
  * README intentionally unchanged otherwise (all other TODOs remain as-is).
"""

with open(sys.argv[1]) as f:
    readme = f.read()

start = "### 📝 Complete TODO List"
end = "## 🤝 Contributing"
idx_start = readme.find(start)
idx_end = readme.find(end)
section = readme[idx_start:idx_end]

LINE_UPDATE = {
    "luffy/deepscaler/utils.py": {
        "Implement OpenAI API client initialization": None,
        "Add proper authentication handling": None,
        "Implement exponential backoff retry logic for rate limits": None,
        "Add comprehensive error handling for different API errors": None,
        "Implement response parsing and validation": None,
        "Add logging for API calls and errors": 45,
        "Support batch processing for multiple prompts": 46,
        "Add timeout configuration for API calls": 47,
        "Implement Vertex AI initialization and authentication": 107,
        "Configure safety settings for content generation": 108,
        "Set up GenerativeModel with proper system instructions": 109,
        "Implement retry logic with exponential backoff": 110,
        "Add comprehensive error handling for API access issues": 111,
        "Handle rate limiting and quota management": 112,
        "Implement response validation and text extraction": 113,
        "Add support for different generation configurations": 114,
    },
    "luffy/verl/verl/protocol.py": {
        "Implement batch dimension folding for efficient processing": None,
        "Add validation for batch size compatibility": None,
        "Handle edge cases where batch_size is not divisible by new_batch_size": None,
        "Optimize memory usage during tensor reshaping": 114,
        "Add support for different tensor types and shapes": 115,
        "Implement batch dimension unfolding functionality": None,
        "Add support for variable batch dimensions": None,
        "Optimize tensor view operations for performance": 136,
        "Handle non-tensor batch data reshaping properly": None,
        "Add error handling for invalid batch dimensions": 137,
        "(zhangchi.usc1992) add consistency check": 169,
        "we can actually lift this restriction if needed": 265,
        "(zhangchi.usc1992) whether to copy": 351,
    },
}

pat = re.compile(r"- \[ \] \*\*(.*?):(\d+)\*\* - (.*)")
items = list(pat.finditer(section))
new_items = []
removed = []
for m in items:
    p = m.group(1)
    l = int(m.group(2))
    d = m.group(3).strip()
    if d in LINE_UPDATE.get(p, {}):
        if LINE_UPDATE[p][d] is None:
            removed.append((p, l, d))
        else:
            new_items.append((p, LINE_UPDATE[p][d], d))
    else:
        new_items.append((p, l, d))

# Sort by file path lexicographic, then line number
from collections import defaultdict
new_items.sort(key=lambda x: (x[0], x[1]))
by_file = defaultdict(list)
for p, l, d in new_items:
    by_file[p].append((l, d))

# Verify ordering
order_ok = True
for f, lst in by_file.items():
    lines = [x[0] for x in lst]
    if lines != sorted(set(lines)):
        order_ok = False
        print(f"ORDER FAIL in {f}: {lines}")
paths_sorted = sorted(by_file)
if list(by_file) != paths_sorted:
    order_ok = False
    print(f"PATH ORDER FAIL: {list(by_file)}")

print("kept:", len(new_items), "removed:", len(removed), "order_ok:", order_ok)
print("Removed:", removed)

new_section = start + "\n\n"
for p, l, d in new_items:
    new_section += f"- [ ] **{p}:{l}** - {d}\n"

readme = readme[:idx_start] + new_section + readme[idx_end:]
with open(sys.argv[1], "w") as f:
    f.write(readme)

print("Updated README TODO section with", len(new_items), "kept items and", len(removed), "removed items")