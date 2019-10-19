If you wish to use an ansible role or other asset in this collection, you should download some or all of this top-level "control center".  To get all of it, use these commands:

  git clone --recurse-submodules https://github.com/abugher/control_center.git
  # ... or, if you have access:
  # git clone --recurse-submodules git@github.com:abugher/control_center.git
  cd control_center
  # I don't understand this part.  Without it I get told each subproject is not on a branch.
  for d in * ansible/roles/*; do cd $d; git checkout master; cd -; done
  g
