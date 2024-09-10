In current wiki folder

git clone wiki_repo

cd wiki_repo

docker run --rm -p 4567:4567 -v $(pwd):/wiki gollumwiki/gollum:latest -d