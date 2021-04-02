# Documentation Update Guide

## Set-up

1. Clone the documentation repository
  - `git clone https://github.com/theicct/roadmap-doc.git`
  - This is a _public_ repository – do not commit any code or sensitive information!
2. Install a Markdown editor
3. Make sure you have access to the latest version of the shared Word document
4. (Optional) Install [Jekyll](https://jekyllrb.com/) if you intend to test the documentation changes locally


## Updating documentation

1. Write your updates in the Word version
2. Get review from the team, as necessary
3. Save as a .pdf file with the name _Roadmap vx.y Model Documentation.pdf_ where _x_ and _y_ reference the model version, e.g. _Roadmap v1.6 Model Documentation.pdf_
4. If you added images to the Word doc, add them to the _assets_ folder
5. Copy your changes to a Markdown version of the documentation either manually or automatically
  - 5a. Copy manually
    - Make a copy of the latest Markdown version of the documentation, using the same naming convention
    - For example, if the latest documentation is `v1.5.md`, create a copy named `v1.6.md` (matching the version of the Roadmap model)
    - Add your changes to the new Markdown file using your favorite Markdown editor
    - If you added images in step 4, add them to the file by inserting the following in the document, where `[filename]` is the name of the image: `![](/roadmap-doc/assets/[filename])`
  - 4b. Copy automatically
    - Edit the file path in _shared_version.sh_ to point to the Word version
    - If you added images in step 4, update the `IMG_MAP` in _doc_to_page.py_
    - Run the _copy_doc.sh_ bash script
6. (Optional) Test your changes locally
  - First, temporarily remove all references to the `roadmap-doc/` directory (it becomes the baseurl once served by GitHub)
  - run `jekyll serve`
7. Add and commit your changes
  - `git commit -am "[Useful description of changes]"`
8. Push your changes, wait a minute, then check to make sure the [online version](https://theicct.github.io/roadmap-doc) updated correctly
