task :default => [:deploy]

task :push do
    system("git push")
end

task :build => :push do
    system("jekyll build")
end

task :deploy => :build do
    HOST = "s158570.gridserver.com"
    USER = "notfromconcentrate.net"
    REMOTE = "domains/pasanpremaratne.com/html/"
    LOCAL = "#{Dir.pwd}/_site/"

    system("rsync -avz --delete #{LOCAL} #{USER}@#{HOST}:#{REMOTE}")
end
