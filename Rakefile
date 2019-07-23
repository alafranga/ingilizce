# frozen_string_literal: true

require 'commonmarker'
require 'json'

class Parser
  def self.call(content)
    JSON.pretty_generate new(content).call
  end

  def initialize(content)
    @doc = CommonMarker.render_doc(content, :DEFAULT)
  end

  def call # rubocop:disable Metrics/MethodLength
    out = {}

    sections(doc).each do |nodes|
      next unless handable?(starting = nodes.shift)

      handler = starting.first_child.string_content.strip.downcase.to_sym

      if Handler.instance_methods(false).include? handler
        out[handler] = send handler, nodes
      else
        warn "No handler for: #{handler}"
      end
    end

    out
  end

  private

  attr_reader :doc

  def error(node, message)
    abort "#{node.sourcepos[:start_line]}: #{node.sourcepos[:start_column]}: #{message}"
  end

  def paragraph_text(node)
    error(node, 'paragraph expected') unless (paragraph = node.first_child).type == :paragraph
    error(node, 'text expected') unless (text = paragraph.first_child).type == :text

    text.string_content
  end

  def handable?(node)
    node.type == :header && node.header_level == 2
  end

  def sections(doc)
    doc.slice_before { |node| handable?(node) }
  end

  module Handler
    def phrases(nodes) # rubocop:disable Metrics/MethodLength
      out = [{}]

      nodes.each do |node|
        case node.type
        when :blockquote
          out.last[:phrase] = paragraph_text(node)
        when :list
          out.last[:clues] = node.map { |item| paragraph_text(item) }
        when :hrule
          out << {}
        end
      end

      out
    end
  end

  include Handler
end

desc 'Create phrases json data from markdown source'
task :phrases do
  File.write('README.json', Parser.call(File.read('README.md')))
end

task default: %i[phrases]
