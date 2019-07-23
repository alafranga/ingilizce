# frozen_string_literal: true

desc 'Create phrases json data from markdown source'
task :phrases do
  sh 'bundle exec bin/phrases-from-markdown <README.md >README.json'
end

task default: %i[phrases]
