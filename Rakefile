task :default => [:deploy]

task :build do
    system("jekyll build")
end

task :deploy => :build do
    HOST = "s158570.gridserver.com"
    USER = "notfromconcentrate.net"
    REMOTE = "domains/pasanpremaratne.com/html/"
    LOCAL = "#{Dir.pwd}/_site/"

    system("rsync -avz --delete #{LOCAL} #{USER}@#{HOST}:#{REMOTE}")
end
