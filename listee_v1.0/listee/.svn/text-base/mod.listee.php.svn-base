<?php

//
// Fellowship Tech >> Listee - EE Module
// Created by Jingyi Wang + Nathan Smith
//

require_once 'f1_config.php';
require_once 'f1_oauth.php';

class Listee {
	// Empty set.
	static private $var = array();

	// Constructor.
	function Listee() {
		if (!count(self::$var)) {
			self::init_f1_setup();
		}
	}

	// Fire up variables.
	function init_f1_setup() {
		$api_consumer = new Oauth_api_client(App_config::$f1_base_url, App_config::$f1_church_code, App_config::$f1_key, App_config::$f1_secret);
		$api_consumer->set_paths_from_config();
		$api_consumer->init_access_token(App_config::$access_token, App_config::$token_secret);

		$var = array();

		$var['api_term'] = isset($_GET['q']) ? htmlentities($_GET['q']) : '';

		if ($var['api_term'] !== '') {
			$var['api_term'] = str_replace(',', ' ', $var['api_term']);
			$var['api_term'] = trim($var['api_term']);
			$var['api_term'] = preg_replace('/\s+/', ' ', $var['api_term']);
			$var['api_term'] = str_replace(' ', ',', $var['api_term']);

			$var['api_show'] = isset($_GET['show']) ? $_GET['show'] : '20';
			$var['api_page'] = isset($_GET['page']) ? $_GET['page'] : '1';

			$request_url = str_replace('{church_code}', App_config::$f1_church_code, App_config::$f1_base_url) . '?searchfor=' . $var['api_term'] . '&recordsperpage=' . $var['api_show'] .'&page=' . $var['api_page'] . '&include=addresses,communications';

			$api_response = $api_consumer->do_request($request_url);

			if ($api_response === '') {
				die('Yikes! Problem communicating with F1.');
			}

			// Output API response.
			$var['json_array'] = json_decode(strstr($api_response, '{"results":{'), true);

			// Results found?
			if (isset($var['json_array']['results']['person'])) {
				$var['json_length'] = count($var['json_array']['results']['person']);
			}

			// Pagination count.
			if ($var['json_array']['results']['@totalRecords'] > 0) {
				$var['total_pages'] = ceil($var['json_array']['results']['@totalRecords'] / $var['api_show']);
			}

			$var['slug'] = '<span class="f1_listee_mute">&mdash;</span>';
		}

		$var['api_term'] = str_replace(',', ' ', $var['api_term']);

		self::$var = $var;
	}

	// {exp:listee:raw_data}
	function raw_data() {
		// Variable bundle.
		$var = self::$var;

		if (isset($var['json_array']) && count($var['json_array']) > 0) {
			return '<pre>' . print_r($var['json_array'], true) . '</pre>';
		}
	}

	// {exp:listee:search_form}
	function search_form() {
		// Variable bundle.
		$var = self::$var;

		$output = <<< HTML
			<form action="" id="f1_listee_form">
				<p>
					<label for="f1_listee_q">
						Search
					</label>
					<input type="text" id="f1_listee_q" name="q" value="{$var['api_term']}" />
					<input type="submit" id="f1_listee_submit" value="Search" />
				</p>
			</form>
HTML;
		return $output;
	}

	// {exp:listee:search_results}
	function search_results() {
		// Variable bundle.
		$var = self::$var;

		$output = '';

		if (isset($var['json_array']) && $var['json_array']['results']['@totalRecords'] > 0) {
			if ($var['total_pages'] > 1) {
				$page_count = 'Page ' . $var['api_page'] . ' of ' . $var['total_pages'];
			}
			else {
				$page_count = '';
			}

			$output .= <<< HTML
				<table id="f1_listee_table" class="rowstyle-alt colstyle-alt sortable-onload-1">
					<caption>
						<span class="f1_listee_float_left">{$var['json_array']['results']['@totalRecords']} results for <strong>{$var['api_term']}</strong></span>
						<span class="f1_listee_mute f1_listee_float_right">$page_count</span>
					</caption>
					<colgroup>
						<col class="f1_listee_col_first_name" />
						<col class="f1_listee_col_last_name" />
						<col class="f1_listee_col_gender"  />
						<col class="f1_listee_col_birthdate" />
						<col class="f1_listee_col_age" />
						<col class="f1_listee_col_phone" />
						<col class="f1_listee_col_address" />
						<col class="f1_listee_col_email" />
					</colgroup>
					<thead>
						<tr>
							<th scope="col" class="sortable-text">
								First
							</th>
							<th scope="col" class="sortable-text">
								Last
							</th>
							<th scope="col" class="sortable-text">
								Gender
							</th>
							<th scope="col" class="sortable-text">
								Birthdate
							</th>
							<th scope="col" class="sortable-numeric">
								Age
							</th>
							<th scope="col" class="sortable-text">
								Phone
							</th>
							<th scope="col" class="sortable-text">
								Address
							</th>
							<th scope="col" class="sortable-text">
								Email
							</th>
						</tr>
					</thead>
HTML;
			if ($var['total_pages'] > 1) {
				$output .= '<tfoot><tr><td colspan="8"><ul>';

				for ($i = 1; $i <= $var['total_pages']; $i++) {
					$output .= '<li><a href="?q=' . $var['api_term'] . '&amp;page=' . $i . '">' . $i . '</a></li>';
				}

				$output .= '<li><a href="?q=' . $var['api_term'] . '&amp;show=' . $var['json_array']['results']['@totalRecords'] . '" class="f1_listee_show_all">Show all</a></li>';

				$output .= '</ul></td></tr></tfoot>';
			}

			$output .= '<tbody>';

			// Drill through array.
			for ($i = 0; $i < $var['json_length']; $i++) {
				$x = $var['json_array']['results']['person'][$i];

				// Person info.
				$first_name = $x['firstName'] ? $x['firstName'] : $var['slug'];
				$last_name = $x['lastName'] ? $x['lastName'] : $var['slug'];
				$gender = $x['gender'] && $x['gender'] !== 'Unspecified' ? $x['gender'] : $var['slug'];
				$birthdate = $x['dateOfBirth'] ? date('M jS, Y', strtotime($x['dateOfBirth'])) : $var['slug'];
				$sort_date = $x['dateOfBirth'] ? date('Y-m-d', strtotime($x['dateOfBirth'])) : $var['slug'];
				$age = $x['dateOfBirth'] ? date('Y') - date('Y', strtotime($x['dateOfBirth'])) : $var['slug'];
				$email = $var['slug'];
				$phone = $var['slug'];

				// Address
				$address1 = $x['addresses']['address'][0]['address1'] ? $x['addresses']['address'][0]['address1'] : '';
				$address2 = $x['addresses']['address'][0]['address2'] ? '<br />' . $x['addresses']['address'][0]['address2'] : '';
				$address3 = $x['addresses']['address'][0]['address3'] ? '<br />' . $x['addresses']['address'][0]['address3'] : '';
				$city = $x['addresses']['address'][0]['city'] ? '<br />' . $x['addresses']['address'][0]['city'] . ' ' : '';
				$state = $x['addresses']['address'][0]['stProvince'] ? $x['addresses']['address'][0]['stProvince'] . ', ' : '';
				$zip = $x['addresses']['address'][0]['postalCode'] ? $x['addresses']['address'][0]['postalCode'] : '';
				$address = $address1 . $address2 . $address3 . $city . $state . $zip;
				$address = $address !== '' ? $address : $var['slug'];

				// Are there comm values?
				$cl = $x['communications']['communication'];

				$email = '';
				$phone = '';

				// Drill through comm values.
				if (count($cl) > 0) {
					for ($j = 0; $j < count($cl); $j++) {
						switch ($cl[$j]['communicationGeneralType']) {
							case 'Telephone':
								$phone .= '<span title="' . $cl[$j]['communicationType']['name'] . '">' . $cl[$j]['communicationValue'] . '</span><br />';
								break;
							case 'Email':
								$email .= '<a href="mailto:' . $cl[$j]['communicationValue'] . '">' . $cl[$j]['communicationValue'] . '</a><br />';
								break;
						}
					}
				}

				// If empty, create slug.
				$email = $email !== '' ? $email : $var['slug'];
				$phone = $phone !== '' ? $phone : $var['slug'];

				$output .= <<< HTML
					<tr>
						<td>$first_name</td>
						<td>$last_name</td>
						<td>$gender</td>
						<td><span class="f1_listee_sort_date">$sort_date</span>$birthdate</td>
						<td>$age</td>
						<td>$phone</td>
						<td>$address</td>
						<td>$email</td>
					</tr>
HTML;
			}

			$output .= '</tbody></table>';
		}
		else if ($var['api_term'] === '') {
			$output .= '<p id="f1_listee_enter_term">Please enter a search term.</p>';
		}
		else {
			$output .= '<p id="f1_listee_no_results">Sorry, no results found for <strong>' . $var['api_term'] . '</strong>.</p>';
		}

		return $output;
	}
}

?>