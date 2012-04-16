{exec} = require 'child_process'

ansi =
	red  			: '\x1B[31m'
	green 			: '\x1B[36m'
	yellow 			: '\x1B[33m'
	blue  			: '\x1B[34m'
	dark_grey  		: '\x1B[1;30m'
	light_grey  	: '\x1B[1;32m'
	reset   		: '\x1B[0m'

log = (message, color) ->
	console.log ansi[color] + message + ansi.reset

option '-m', '--message [COMMIT_MESSAGE]', 'set git commit message'
option '-t', '--title [POST / PAGE TITLE]', 'set title for new page / post'

task 'save', 'commit source then generate and deploy site', (options) ->
	message = options.message || 'minor change'
	exec 'git add -f .;', (err, stdout, stderr) ->
		log stdout + stderr, 'light_grey'
		err && log err, 'red'
		exec 'git commit -m "' + message + '"', (err, stdout, stderr) ->
			log stdout + stderr, 'light_grey'
			err && log err, 'red'
			exec 'git push origin source', (err, stdout, stderr) ->
				log stdout + stderr, 'light_grey'
				err && log err, 'red'
				# below use 'bundle exec rake...' due to version mismatch
				exec 'bundle exec rake generate', (err, stdout, stderr) ->
					log stdout + stderr, 'light_grey'
					err && throw err
					exec 'bundle exec rake deploy', (err, stdout, stderr) ->
						log stdout + stderr, 'light_grey'
						err && throw err
						log 'Save OK!', 'green'

###
task 'post', 'create new post', (options) ->
	exec 'bundle exec rake new_post["' + options.title + '"]', (err, stdout, stderr) ->
		log stdout + stderr, 'light_grey'
		err && throw err
		log 'Post created!', 'green'

