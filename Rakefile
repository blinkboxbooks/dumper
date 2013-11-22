desc "Move messages from a rabbit queue to an exchange on the same server"
task :shunt => :amqp_connect do
  require 'yaml'

  profiles = YAML.load(open('profiles.yaml'))

  profile = profiles[ARGV[1]]

  if profile.nil?
    $stderr.puts "There is no profile called '#{ARGV[1]}' in profiles.yaml"
    exit(false)
  end

  read_channel = @connection.create_channel
  read_channel.prefetch(@properties[:prefetch] || 1)

  write_channel = @connection.create_channel

  queue = read_channel.queue(profile['read']['queue_name'],profile['read']['options'])
  puts "Connected to Queue: #{profile['read']['queue_name']}"

  if queue.message_count == 0
    puts "No messages on the queue! Job's a good'un."
    exit(true)
  end

  puts "  #{queue.message_count} messages to process"

  exchange = write_channel.send(profile['write']['type'].to_sym,profile['write']['exchange_name'],profile['write']['options'])
  puts "Connected to Exchange: #{profile['write']['exchange_name']}"

  queue.subscribe(:ack => true, :block => true) do |delivery_info, metadata, payload|
    begin
      exchange.publish(payload,:persistent => true,:nowait => false)
      read_channel.acknowledge(delivery_info.delivery_tag, false)

      if queue.message_count == 0
        puts "All messages processed! Job's a good'un."
        exit(true)
      end
    rescue StandardError => e
      $stderr.puts "Failed to move a message. I quit, sending the message back to the original queue"
      raise e
      exit(false)
    end
  end

  # Don't attempt to run the arguments as rake tasks!
  exit(true)
end

task :properties do
  # Find a suitable properties file
  propfile = "rabbit_shunt.properties"
  unless File.exist?(propfile)
    $stderr.puts "No properties file found."
    exit(false)
  end
  require 'java_properties'
  @properties = JavaProperties::Properties.new(propfile)
end

task :amqp_connect => :properties do
  require 'bunny'
  @connection = Bunny.new(@properties[:mq])
  @connection.start
end

