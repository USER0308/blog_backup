pip install pylint

pip install pylint-django

pycharm->file->settings->tools->External tools
+
Program = /usr/local/bin/pylint
Parameters = --load-plugins pylint_django -rn --msg-template="{abspath}:{line}: [{msg_id}({symbol}), {obj}] {msg}" $FilePath$
Working directory = $FileDir$
